# SHAP 图表问题

<show-structure depth="3"/>


## 1. 添加标题

使用 `shap.summary_plot` 函数绘制特征重要性时，自带的图表是无法设置标题的，但是我们可以借助 matplotlib 库来完整。

```Python
import shap

explainer = Explainer(lr, df[features], feature_names=features)
shap_values = explainer(df[features])
shap.summary_plot(
    shap_values, max_display=30, plot_type="bar", show=False
    )
fig = plt.gcf()
fig.set_size_inches(12, 8)
plt.title("Feature Importance by LR SHAP Explanation")
plt.show()
```

需要注意的是，一定将其 `summary_plot` 函数中 `show` 参数设置为 `False`，所设置的标题才会生效。

