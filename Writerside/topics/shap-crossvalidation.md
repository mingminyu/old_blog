# 交叉验证下的 SHAP 解释

<show-structure depth="3"/>

> 摘录[在Python中使用交叉验证进行SHAP解释](https://mp.weixin.qq.com/s/JwJUkAxx6rEYKVTcuVD6iw) 仅供自己学习，请大家点击原文查看。

## 1. 背景介绍

在许多情况下，由于其出色的预测性能和处理复杂非线性数据的能力，机器学习模型通常优于传统的线性模型。然而，机器学习模型常见的批评是它们缺乏可解释性。例如，集成方法如 XGBoost 和随机森林将许多个学习器的结果结合起来生成它们的结果。尽管这通常导致更好的性能，但它使得很难知道数据集中每个特征对输出的贡献是多少。


为了解决这个问题，可解释的人工智能（xAI）已经被提出并越来越受欢迎。xAI 领域旨在解释这些不可解释的模型（所谓的黑匣子模型）是如何进行预测的，从而实现了预测准确性和可解释性的最佳结合。这背后的动机是，许多机器学习的实际应用不仅需要良好的预测性能，还需要解释结果生成的方式。例如，在医学领域，根据模型做出的决策可能导致生命的丧失或挽救，因此了解决策的驱动因素非常重要。此外，能够识别重要的变量可以为识别机制或治疗途径提供信息。


其中最受欢迎和有效的 xAI 技术之一是 SHAP，SHAP 概念于 2017 年由 Lundberg 和 Lee 引入，但实际上是建立在游戏理论中的 Shapley 值之上，这个概念早在此之前就存在了。简而言之，SHAP 值通过计算每个特征的边际贡献来工作，方法是在许多带有该特征和不带该特征的模型的预测（每个观察）中查看这种贡献，权衡这些减少的特征集在模型中的贡献，然后将所有这些实例的加权贡献相加。需要更详细描述的人可以参考上面的链接，但对于我们的目的来说，简单地说：**观察的 SHAP 值的绝对值越大，对预测的影响就越大。因此，对于给定特征的所有观察的绝对 SHAP 值的平均值越大，该特征就越重要**。


在 Python 中实现 SHAP 值非常容易，使用 SHAP 库，并且在线上已经存在许多解释如何做到这一点的教程。然而，我在所有的指南中都发现了两个主要不足之处。

首先，大多数文档都在基本的训练/测试拆分上使用 SHAP 值，而不是在交叉验证上使用（见图1）。使用交叉验证可以更好地了解结果的泛化能力，而简单的训练/测试拆分的结果可能会根据数据的分割方式而发生较大变化。正如我在我的最新文章“营养研究中的机器学习”中解释的那样，**除非你处理的数据集非常庞大，否则几乎总是应该优先使用交叉验证，而不是拆分训练/测试**。

![](%myimgs%shap_crossvalidation_1.png?raw=true)

另一个不足之处是，我所找到的所有文档**都没有使用多次重复的交叉验证来计算它们的 SHAP 值**。虽然交叉验证在简单的训练/测试拆分上是一个重大进步，但最好的做法是使用不同的数据拆分多次重复进行交叉验证。这在数据较小的情况下尤为重要，因为结果可能会根据数据的拆分方式而发生很大变化。这就是为什么通常建议重复 100 次交叉验证以确保结果的可信度。

为了解决这些不足之处，我决定编写一些代码来自己实现这一点。本教程将向你展示如何获得多次交叉验证的 SHAP 值，并结合嵌套交叉验证方案。对于我们的模型数据集，我们将使用波士顿房价数据集，并选择强大但不可解释的随机森林算法。

## 2. SHAP值的实施

每当你构建带有各种循环的代码时，通常最好从最内部的循环开始，然后向外部扩展。尝试从外部开始并按照代码将运行的顺序构建代码会更容易混淆，当事情出错时也更难排除故障。

无论何时，当你构建带有各种循环的代码时，通常最好从最内部的循环开始，然后向外部扩展。通过尝试从外部开始构建代码，并按照代码将运行的顺序构建，更容易混淆，并且在出现问题时更难进行故障排除。

因此，我们从 SHAP 值的基本实现开始。我会假设你熟悉 SHAP 的一般用法以及其实现代码的外观，因此我不会花太多时间进行解释。我在整个代码中都留下了注释（这是一种常见的做法），所以你可以查看这些注释，如果你仍然不确定，可以查看引言中的链接或库的文档。我还是根据需要逐个导入库，而不是一次性全部导入，这有助于理解。

```Python
import shap
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

url = 'https://raw.githubusercontent.com/Sketchjar/MachineLearningHD/main/boston_data.csv'
df = pd.read_csv(url)
df.drop('Unnamed: 0', axis=1, inplace=True)
X, y = df.drop('Target', axis=1), df.Target

# Split data, establish model, fit model, 
# make prediction, score model, print result
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=10
    )
# Random state for reproducibility (same results every time)
model = RandomForestRegressor(random_state=10)
fit = model.fit(X_train, y_train)
yhat = fit.predict(X_test)
result = mean_squared_error(y_test, yhat)
print('RMSE:', round(np.sqrt(result), 4))

# Use SHAP to explain predictions
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)
shap.summary_plot(shap_values, features = X.columns)
```

![](%myimgs%shap_crossvalidation_2.png?raw=true)

## 3. 将交叉验证与SHAP值结合

通常，我们习惯于使用 sklearn 的 `cross_val_score` 或类似的自动方式实现交叉验证。但这种方式的问题是一切都在幕后发生，我们无法访问每个折叠中的数据。当然，如果我们想要获取所有数据点的 SHAP 值，我们需要访问每个数据点（请记住，每个数据点在测试集中仅使用一次，在训练中使用 k-1 次）。为了解决这个问题，我们可以将 KFold 与 `.split` 结合使用。


```Python
from sklearn.model_selection import KFold

# Establish CV scheme
CV = KFold(n_splits=5, shuffle=True, random_state=10)

ix_training, ix_test = [], []
# Loop through each fold and append the training 
# and test indices to the empty lists above
for fold in CV.split(df):
    ix_training.append(fold[0]), ix_test.append(fold[1])
```

通过使用 `.split` 循环遍历我们的 `KFold` 对象，我们可以获取每个折叠的训练和测试索引。这里 `fold` 是一个元组，`fold[0]` 是每个折叠的训练索引，`fold[1]` 是测试索引。

现在，我们可以使用这个信息自己从原始数据帧中选择训练和测试数据，从而提取我们想要的信息。我们通过创建一个新的 `for` 循环来做到这一点，以获取每个折叠的训练和测试索引，然后像平常一样执行我们的回归和 SHAP 过程。然后，我们只需要在循环外添加一个空列表，以跟踪每个样本的SHAP值，然后在循环结束时将这些值附加到列表中。我使用“#-#-#”来表示这些新的添加部分：

```Python
SHAP_values_per_fold = [] #-#-#

## Loop through each outer fold and extract SHAP values 
for i, (train_outer_ix, test_outer_ix) in enumerate(zip(ix_training, ix_test)): #-#-#
    # Verbose
    print('\n------ Fold Number:',i)
    X_train, X_test = X.iloc[train_outer_ix, :], X.iloc[test_outer_ix, :]
    y_train, y_test = y.iloc[train_outer_ix], y.iloc[test_outer_ix]
    
    # Random state for reproducibility (same results every time)
    model = RandomForestRegressor(random_state=10) 
    fit = model.fit(X_train, y_train)
    yhat = fit.predict(X_test)
    result = mean_squared_error(y_test, yhat)
    print('RMSE:', round(np.sqrt(result), 4))

    # Use SHAP to explain predictions
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_test)
    for SHAPs in shap_values:
        SHAP_values_per_fold.append(SHAPs) #-#-#
```

现在，我们对每个样本都有了 SHAP 值，而不仅仅是数据的一个测试拆分中的样本，并且我们可以使用 SHAP 库轻松绘制这些值。我们只需要首先更新 `X` 的索引，以匹配它们在每个折叠的每个测试集中出现的顺序，否则，颜色编码的特征值将全部错误。请注意，在 `summary_plot` 函数内部，我们重新排列 `X`，以便不保存更改到原始 `X` 数据帧中：


```Python
new_index = [ix for ix_test_fold in ix_test for ix in ix_test_fold]
shap.summary_plot(np.array(SHAP_values_per_fold), X.reindex(new_index))
```

![](%myimgs%shap_crossvalidation_3.png?raw=true)

从图中可以看出，与仅使用训练/测试拆分相比，现在有更多的数据点（事实上，所有数据点）。这已经改善了我们的过程，因为我们可以使用整个数据集，而不仅仅是一部分。

但我们仍然不清楚稳定性，即如果数据拆分方式不同，结果会如何变化。幸运的是，我们可以通过以下代码来解决这个问题。


## 4. 重复交叉验证

使用交叉验证大大增加了工作的稳健性，特别是对于较小的数据集。然而，如果我们真的想做好数据科学，那么交叉验证应该在数据的许多不同拆分上重复进行。

首先，我们现在需要考虑的不仅是每个折叠的 SHAP 值，还有每个重复的每个折叠的 SHAP 值，然后将它们合并到一个图中绘制。字典在 Python 中是强大的工具，这就是我们将使用它来跟踪每个样本在每个折叠中的 SHAP 值的原因。

首先，我们决定要执行多少次交叉验证重复，并建立一个字典来存储每个样本在每次重复中的 SHAP 值。通过循环遍历我们数据集中的所有样本，并在我们的空字典中为它们创建一个键，然后在每个样本内部创建另一个键来表示交叉验证重复。

```Python
np.random.seed(1) # Reproducibility 
CV_repeats = 10

# Make a list of random integers between 0 and 10000 of length = CV_repeats 
#   to act as different data splijts
random_states = np.random.randint(10000, size=CV_repeats) 

######## Use a dict to track the SHAP values of each observation per CV repitition 

shap_values_per_cv = dict()
for sample in X.index:
    ## Create keys for each sample
    shap_values_per_cv[sample] = {} 
    
    ## Then, keys for each CV fold within each sample
    for CV_repeat in range(CV_repeats):
        shap_values_per_cv[sample][CV_repeat] = {}
```

然后，我们在现有代码中添加一些新行，允许我们重复进行 `CV_repeats` 次交叉验证过程，并将每次重复的SHAP值添加到我们的字典中。这很容易实现，只需更新代码末尾的一些行，以便不是将 SHAP 值的列表附加到列表中，而是更新字典。（注意：收集每个折叠的测试分数可能也是相关的，尽管我们在这里没有这样做，因为重点是使用 SHAP 值，但可以通过添加另一个字典，将CV重复作为键，测试分数作为值，轻松进行更新）。


以下是代码示例，其中“#-#-#”表示对现有代码的更新部分：

```Python
for i, CV_repeat in enumerate(range(CV_repeats)): #-#-#
    #Verbose 
    print('\n------------ CV Repeat number:', CV_repeat)
    #Establish CV scheme
    CV = KFold(n_splits=5, shuffle=True, random_state=random_states[i]) # Set random state 

    ix_training, ix_test = [], []
    # Loop through each fold and append the training & test indices to the empty lists above
    for fold in CV.split(df):
        ix_training.append(fold[0]), ix_test.append(fold[1])

    ## Loop through each outer fold and extract SHAP values 
    for i, (train_outer_ix, test_outer_ix) in enumerate(zip(ix_training, ix_test)): 
        #Verbose
        print('\n------ Fold Number:',i)
        X_train, X_test = X.iloc[train_outer_ix, :], X.iloc[test_outer_ix, :]
        y_train, y_test = y.iloc[train_outer_ix], y.iloc[test_outer_ix]

        model = RandomForestRegressor(random_state=10) # Random state for reproducibility (same results every time)
        fit = model.fit(X_train, y_train)
        yhat = fit.predict(X_test)
        result = mean_squared_error(y_test, yhat)
        print('RMSE:',round(np.sqrt(result),4))

        # Use SHAP to explain predictions
        explainer = shap.TreeExplainer(model)
        shap_values = explainer.shap_values(X_test)

        # Extract SHAP information per fold per sample 
        for i, test_index in enumerate(test_outer_ix):
            shap_values_per_cv[test_index][CV_repeat] = shap_values[i] #-#-#
```


要可视化这些数据，假设我们想要检查第五次交叉验证重复中索引号为 10 的样本，我们只需写：

```Python
shap_values_per_cv[10][5]
# Returns
'''
array([ 1.07964272,  0.03934705, -0.04219157, -0.03962084,  1.0635659 ,
       -1.9630717 , -0.25090775,  0.31162461,  0.01585746,  0.13389457,
        0.69374553,  0.25784941, -3.50476256])
'''
```

其中第一个方括号表示样本编号，第二个表示重复次数。输出是第五次交叉验证重复后样本编号为 10 的每列 X 的 SHAP 值。

要查看一个个体的所有交叉验证重复的 SHAP 值，我们只需在第一个方括号中输入编号：

```Python
shap_values_per_cv[400]
# Returns 
'''
{0: array([-1.24861892e+00, -5.49884230e-03,  5.79160167e-02, -8.83081945e-03,
        -1.45949631e+00, -3.18839888e+00, -1.39426397e-01, -5.51482918e-01,
         7.76179546e-02, -1.76992935e-01, -2.51426566e-01, -7.43384863e-02,
        -7.44069864e+00]),
 1: array([-1.58130797e+00, -5.12671026e-03,  2.74038359e-02, -7.03857730e-03,
        -1.39333035e+00, -1.72330227e+00, -1.01255310e-01, -1.13856269e-01,
         1.14530229e-02, -1.54180290e-01, -2.75701360e-01, -2.35938425e-02,
        -9.12103767e+00]),
 2: array([-1.41873698e+00, -2.54806852e-03,  4.71476426e-02, -3.58564189e-03,
        -1.51096119e+00, -3.77213917e+00, -3.11641853e-01, -4.06127783e-02,
         2.99869569e-02, -1.77258135e-01, -2.97570318e-01,  5.50327430e-02,
        -6.68247371e+00]),
 3: array([-1.05989830e+00, -1.44528209e-03,  1.46173653e-02, -7.39191941e-03,
        -1.36852214e+00, -2.80498851e+00,  3.73204626e-02, -4.10953637e-01,
         2.51015011e-02, -2.36520416e-01, -4.55250409e-01,  9.81373566e-02,
        -8.13198631e+00]),
 4: array([-5.37416841e-01, -3.72063436e-03,  1.62621596e-02, -6.63224124e-03,
        -1.25173037e+00, -2.70076639e+00, -2.84206491e-01, -1.01482421e+00,
         1.23694105e-01, -1.69387679e-01, -3.09232302e-01,  1.91462402e-01,
        -7.19429162e+00]),
 5: array([-1.41994259, -0.01382437,  0.0259338 , -0.00933146, -1.43983385,
        -3.15449993, -0.15794043, -0.19650086,  0.02529137, -0.24128899,
        -0.41353906, -0.05474904, -7.25665579]),
 6: array([-1.77178487e+00,  4.88349913e-04,  6.56877260e-02, -1.91605214e-02,
        -1.34849438e+00, -2.40857273e+00, -2.44969830e-01, -6.59495940e-01,
         1.53839141e-01, -3.36269323e-01, -2.80693205e-01,  3.83342527e-02,
        -7.41086412e+00]),
 7: array([-1.20049002e+00, -1.22261858e-02,  6.28946517e-02, -7.57665474e-03,
        -1.49350196e+00, -1.92901554e+00, -8.92221397e-02, -6.09658522e-01,
         5.09614503e-02, -2.98475317e-01, -2.31665455e-01,  3.93005676e-02,
        -8.09737686e+00]),
 8: array([-1.92971806e+00, -1.75587118e-03,  7.24273666e-02, -4.03754018e-03,
        -1.21828980e+00, -3.69425705e+00, -1.89533662e-01, -1.87271694e-01,
         4.34000535e-02, -1.14723010e-01, -3.35681375e-01, -7.54959315e-02,
        -7.05037453e+00]),
 9: array([-1.11032383e+00, -2.54275839e-03,  5.02963229e-04, -1.14619478e-02,
        -1.28660325e+00, -2.08989776e+00, -9.17563771e-02, -4.53554446e-01,
         5.16116092e-02, -1.66085331e-01, -3.40811440e-01,  6.99992394e-02,
        -8.46009148e+00])}
'''
```
{collapsible="true" default-state="expanded"}

但这对我们来说并没有多大用处（除了用于故障排除目的），我们真正需要的是绘制图表来可视化这些数据。

首先，我们需要将每个样本每个交叉验证重复的 SHAP 值平均为一个值以进行绘制（如果你愿意，还可以使用中位数或其他统计数据）。平均值很方便，但可能会隐藏数据内部的变异性，这也可能是需要了解的。因此，在我们计算平均值的同时，我们还将获得其他统计数据，如最小值、最大值和标准差：

```Python
# Establish lists to keep average Shap values, their Stds, and their min and max
average_shap_values, stds, ranges = [],[],[]

for i in range(0,len(df)):
    # Get all SHAP values for sample number i
    df_per_obs = pd.DataFrame.from_dict(shap_values_per_cv[i])
    # Get relevant statistics for every sample 
    average_shap_values.append(df_per_obs.mean(axis=1).values) 
    stds.append(df_per_obs.std(axis=1).values)
    ranges.append(df_per_obs.max(axis=1).values - df_per_obs.min(axis=1).values)
```

上面的代码表示：对于我们原始数据帧中的每个样本索引，创建一个数据帧，其中包含每个 SHAP 值列表（即每个交叉验证重复）。该数据帧将每个交叉验证重复作为一行，每个 `X` 变量作为一列。现在，我们使用适当的函数并使用 `axis=1` 来对每列进行平均、标准差、最小值和最大值的计算，然后将每个值转换为数据帧。

现在，我们只需像绘制常规值一样绘制平均值。我们在这里也不需要重新排序索引，因为我们从字典中获取 SHAP 值，而字典的顺序与 X 的顺序相同。


```Python
shap.summary_plot(np.array(average_shap_values), X, show=False)
plt.title('Average SHAP values after 10x cross-validation')
```

![](%myimgs%shap_crossvalidation_4.png?raw=true)


由于我们的结果已经在多次重复的交叉验证中进行了平均，因此它们比仅执行一次的简单训练/测试拆分更稳健且可信。

但是，如果你将绘图前后的图表进行比较，并发现除了额外的数据点外，几乎没有变化，那么你可能会感到失望。但不要忘记，我们使用的是一个模型数据集，该数据集非常整洁，具有与结果之间的强关系。在不太理想的情况下，像重复的交叉验证这样的技术将揭示实际数据在结果和特征重要性方面的不稳定性。

如果我们想进一步加强我们的结果（当然我们想这样做），我们可以添加一些图表，以了解我们提出的特征重要性的变异性。这是相关的，因为计算每个样本的平均 SHAP 值可能会掩盖它们在不同数据拆分下的变化程度。

为此，我们必须将我们的数据帧转换为长格式，之后我们可以使用 seaborn 库创建一个 catplot。

```Python
import seaborn as sns
from matplotlib import pyplot as plt
ranges = pd.DataFrame(ranges)
ranges.columns = X.columns

# Transpose dataframe to long form
values, labels = [],[]
for i in range(len(ranges.columns)):
    for j in range(len(ranges)):
        values.append(ranges.T[j][i])
        labels.append(ranges.columns[i])

long_df = pd.DataFrame([values,labels]).T
long_df.columns = ['Values', 'Features']

title = 'Range of SHAP values per sample across all\ncross-validation repeats'
xlab, ylab = 'SHAP Value Variability', 'SHAP range per sample'
sns.catplot(data=long_df, x='Features', y='Values').set(
            xlabel=xlab, ylabel=ylab, title=title)
plt.xticks(rotation=45)
```

![](%myimgs%shap_crossvalidation_5.png?raw=true)

在上面的 catplot 中，我们看到了每个样本的每个交叉验证重复的范围（最大值减去最小值）。理想情况下，我们希望Y轴上的值尽可能小，因为这意味着更一致的特征重要性。

然而，我们应该记住，这种变异性也对绝对特征重要性敏感，即被认为更重要的特征自然会有具有更大范围的数据点。我们可以通过对数据进行缩放来部分考虑这一点。


```Python
mean_abs_effects = long_df.groupby(['Features']).mean()

standardized = long_df.groupby(long_df.Features).transform(lambda x: x/x.mean())
standardized['Features'] = long_df.Features

title = 'Scaled Range of SHAP values per sample \nacross all cross-validation repeats'
sns.catplot(data=standardized, x='Features', y='Values').set(
    xlabel='SHAP Value Variability Scaled by Mean', title=title
    )
plt.xticks(rotation=45)
```

![](%myimgs%shap_crossvalidation_6.png?raw=true)


请注意，LSTAT 和 RM 两个我们最重要的特征的情况看起来不同。现在，我们更好地反映了按特征的整体重要性进行缩放的变异性，这取决于我们的研究问题，这可能更相关或不相关。

我们可以根据我们收集的其他统计信息，例如标准差等，制作类似的图表。


## 5. 嵌套交叉验证

这一切都很棒，但还有一件事缺失：我们的随机森林处于其默认模式下。尽管默认参数在这个数据集上表现相当不错，但在其他情况下可能不是这样。而且，为什么我们不尝试最大化我们的结果呢？

我们应该注意，不要陷入一个在当今的机器学习示例中似乎非常普遍的陷阱，即在优化模型的超参数时，也在测试集中存在数据。通过简单的训练/测试拆分，可以轻松避免这种情况，只需在训练数据上优化超参数即可。

但是一旦引入了交叉验证，这个概念似乎就被忘记了。实际上，人们经常使用交叉验证来优化超参数，然后使用交叉验证来评分模型。在这种情况下，数据泄漏已经发生，我们的结果将会（即使只有轻微的）过于乐观。

嵌套交叉验证是我们应对这个问题的解决方案。它涉及采用我们正常的交叉验证方案中的每个训练折叠（这里称为“外循环”），通过在每个折叠的训练数据上使用另一个交叉验证（称为“内循环”）来优化超参数。这意味着我们在训练数据上优化超参数，然后仍然可以对优化后的模型在未见数据上的性能有一个较少偏见的想法。

这个概念可能有点难以理解，但对于那些希望了解更多细节的人，我在上面链接的文章中有解释。无论如何，这段代码并不难，通过阅读它可能有助于理解。事实上，我们在上面的过程中已经准备好了大部分代码，只需要进行小的调整。让我们看看它是如何运作的。

嵌套交叉验证的主要考虑因素，特别是在我们使用许多重复的情况下，它需要花费大量时间来运行。因此，我们将保持参数空间较小，并使用随机搜索而不是网格搜索（尽管在大多数情况下，随机搜索通常在大多数情况下表现得足够好）。如果你想更加彻底，可能需要在高性能计算机上保留一些时间。无论如何，在我们的初始for循环之外，我们将建立参数空间：

```Python
param_grid = [{
    'max_depth': [10, 20, 30],
    'min_samples_leaf': [1, 2],
    'min_samples_split': [2, 5],
    'n_estimators': [200, 800], 
    'ccp_alpha': np.linspace(0.1, 1)
   }]
```

然后，我们对原始代码进行以下更改：

CV 现在将变为 `cv_outer`，因为现在我们有两个交叉验证，我们需要适当地引用每个交叉验证。

在我们的 for 循环中，我们循环遍历训练和测试 ID 时，我们添加了我们的内部交叉验证方案 `cv_inner`。

然后，我们使用 `RandomizedSearchCV` 来优化我们的模型在 `inner_cv` 上，选择最佳模型，然后使用最佳模型从测试数据中提取 SHAP 值（这里的测试数据是外部折叠测试）。

就是这样。出于演示目的，我们将 `CV_repeats` 减小到 2，因为否则，可能需要很长时间。在实际情况下，你需要保持足够高的 `CV_repeats` 以保持具有最佳参数的健壮结果，这可能不需要高性能计算机（或者需要耐心）。

查看以下代码以获取这些更改，再次使用“#-#-#”表示新添加的部分。

```Python
CV_repeats = 2
from sklearn.model_selection import RandomizedSearchCV

for i, CV_repeat in enumerate(range(CV_repeats)): 
    #Verbose 
    print('\n------------ CV Repeat number:', CV_repeat)
    #Establish CV scheme
    CV = KFold(n_splits=5, shuffle=True, random_state=random_states[i]) # Set random state 

    ix_training, ix_test = [], []
    # Loop through each fold and append the training & test indices to the empty lists above
    for fold in CV.split(df):
        ix_training.append(fold[0]), ix_test.append(fold[1])

    ## Loop through each outer fold and extract SHAP values 
    for i, (train_outer_ix, test_outer_ix) in enumerate(zip(ix_training, ix_test)): 
        #Verbose
        print('\n------ Fold Number:',i)
        X_train, X_test = X.iloc[train_outer_ix, :], X.iloc[test_outer_ix, :]
        y_train, y_test = y.iloc[train_outer_ix], y.iloc[test_outer_ix]

        ## Establish inner CV for parameter optimization #-#-#
        cv_inner = KFold(n_splits=3, shuffle=True, random_state=1) #-#-#

        # Search to optimize hyperparameters
        model = RandomForestRegressor(random_state=10)
        search = RandomizedSearchCV(model, param_grid, scoring='neg_mean_squared_error', cv=cv_inner, refit=True) #-#-#
        result = search.fit(X_train, y_train) #-#=#

        # Fit model on training data 
        result.best_estimator_.fit(X_train, y_train) #-#-#

        # Use SHAP to explain predictions using best estimator 
        explainer = shap.TreeExplainer(result.best_estimator_) 
        shap_values = explainer.shap_values(X_test)

        # Extract SHAP information per fold per sample 
        for i, test_index in enumerate(test_outer_ix):
            shap_values_per_cv[test_index][CV_repeat] = shap_values[i] 
```

## 6. 结论

解释复杂的 AI 模型的能力变得越来越重要，SHAP 值是实现这一目标的一种很好的方式，然而，单个训练/测试拆分的结果并不总是可信的，特别是在较小的数据集中。通过多次重复程序，如（嵌套）交叉验证，你可以提高结果的稳健性，并更好地估计如果底层数据也发生变化，你的结果可能会如何改变。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/JwJUkAxx6rEYKVTcuVD6iw">在 Python 中使用交叉验证进行SHAP解释</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>