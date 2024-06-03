# 进阶使用

<show-structure depth="3"/>

## 1.  配置 YAML 文件自动补全

我们都知道在 IDEA 中构建 SpringBoot 项目，在编写 .properties 或者 .yml 配置文件时，IDEA 可以提供参数提示以及自动补全的功能，这使得整个项目的搭建和维护都变得非常简单。但是对于使用 Python 的同学都知道，PyCharm 中并没有相应的插件实现此功能，尤其是我们自行搭建一些框架时，使用 YAML 作为配置文件中所使用的参数各有不同，这就导致框架的使用变得比较困难。


万幸的是，我们可以通过 PyCharm 中 JSON Schema Mappings 功能可以达成预期的目的，从而能在使用自定义框架或者一些外部框架时，在配置 YAML 文件可以自动补全参数。接下来，我们看下在 PyCharm 中如何实现。


### 1.1 自定义 Json Scheme 文件

现在我们定义一个 JSON Schema，内容为需要自动补全的参数名称、类型以及值，保存文件内容为 demo.json。

```JSON
{
  "$id": "https://example.com/blueprint.schema.json",
  "$schema": "http://json-schema.org/draft-08/schema#",
  "title": "Blueprint",
  "description": "Cloudify Blueprint",
  "type": "object",
  "properties": {
    "tosca_definitions_version": {
      "type": "string",
      "enum": [
        "cloudify_dsl_1_0",
        "cloudify_dsl_1_1",
        "cloudify_dsl_1_2",
        "cloudify_dsl_1_3"
      ]
    },
    "imports": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "inputs": {
      "type": "object",
      "properties": {
        "webserver_port": {
          "type": "object",
          "properties": {
            "description": {
              "type": "string"
            },
            "type": {
              "type": "string"
            },
            "default": {
              "type": "number"
            }
          }
        }
      }
    },
    "node_templates": {
      "type": "object",
      "patternProperties": {
        "": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string"
            },
            "properties": {
              "type": "object"
            },
            "interfaces": {
              "type": "object",
              "properties": {
                "create": {
                  "$ref": "#/definitions/interfaceAction"
                },
                "delete": {
                  "$ref": "#/definitions/interfaceAction"
                }
              }
            }
          }
        }
      }
    },
    "definitions": {
      "interfaceAction": {
        "type": "object",
        "properties": {
          "implementation": {
            "type": "string"
          }
        }
      }
    }
  }
}
```

> 关于更多的参数配置形式，可以参考 [SchemaStore](https://www.schemastore.org/json) 中的模板文件来仿写。
> 
{style="note"}

### 1.2 在 PyCharm 中完成配置

操作步骤:
1. 在 PyCharm 中找到 Settings | Languages & Frameworks | Schemas and DTDs | JSON Schema Mappings 设置选项
2. 定义一个名称 demo yaml，名称可以根据实际情况自己修改
3. 在 **Schema file or URL** 中填入我们保存的 demo.json 文件路径
4. 点击 **+** 号新增一项，可以选择文件，也可以选择 Add File Path Pattern，作用是限定自动参数补全的文件名 Pattern。

> 当我们有多个项目框架需要导入进行自动补全时时，那么我们最好定义好 File Path Pattern 来限定作用的文件名称，比如
> 
{style="warning"}


### 1.3 快速创建配置信息

完成配置后，我们就可以创建 test.yml 文件，试试利用自动补全的功能快速完成下面的参数配置。

```yaml
tosca_definitions_version: cloudify_dsl_1_3 
imports:  
  - http://cloudify.co/spec/cloudify/5.0.5/types.yaml 
inputs:   
  webserver_port:    
    description: The HTTP web server port.    
    default: 8000 
node_templates:
   
  http_web_server:    
    type: cloudify.nodes.WebServer    
    properties:      
      port: { get_input: webserver_port }    
    interfaces:      
      cloudify.interfaces.lifecycle:        
        create:           
          implementation: install.py          
          executor: central_deployment_agent        
        delete:           
          implementation: uninstall.py          
          executor: central_deployment_agent
```





<seealso>
<category ref="ref_docs">
    <a href="https://medium.com/@alexmolev/boost-your-yaml-with-autocompletion-and-validation-b74735268ad7">PyCharm 配置 YAML 文件自动补全</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>