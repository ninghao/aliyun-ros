## 相关链接
- 帮助文件：https://help.aliyun.com/document_detail/28852.html
- 资源类型：https://ros.console.aliyun.com/#/resourceType/list
- 模板示例：https://ros.console.aliyun.com/#/template/list
- 社区博客：https://yq.aliyun.com/teams/12?spm=5176.2020520146.103.4.tbKhSc
- RDS 实例规格表：https://help.aliyun.com/document_detail/26312.html

## 限制
- 每个堆栈允许创建的最大资源数为 200 个
- 每个用户允许创建的堆栈数最大为 50 个
- 每个模板文件的大小不超过　512KB

## 模板
```
ROSTemplateFormatVersion: '2015-09-01'
Description: '描述模板的作用'
Parameters:
Mappings:
Resources:
Outputs:
```

### ROSTemplateFormatVersion（必需）
模板格式的版本号，目前只能是：2015-09-01 。

### Description（可选）
描述一下模板的作用。

### Parameters（可选）
参数，创建堆栈时可以指定参数的值，然后在模板的 Resources 和 Outputs 里可以引用参数的值。

#### 语法

- Type：数据类型，可以是 String，Number，CommaDelimitedList，Json，Boolean
- Default：默认值
- AllowedValues：允许的值的列表
- AllowedPattern：使用正则表达式设置允许的值的模式
- MaxLength：最大长度
- MinLength：最小长度
- MaxValue：最大值
- MinValue：最小值
- NoEcho：调用查询堆栈时是否输出参数值。如果将值设置为 true，则只输出星号 (\**)
- Description：描述
- ConstraintDescription：不符合约束条件时显示的提示信息

#### 示例
```
Parameters:
  EcsType:
    Type: 'String'
    Default: 'ecs.t1.small'
    AllowedValues:
      - 'ecs.t1.small'
      - 'ecs.s1.medium'
      - 'ecs.m1.medium'
      - 'ecs.c1.large'
    Description: '选择 ECS 配置。'
```

编排服务允许用户传入 EcsType 作为参数。如果不传，将使用默认的参数即 ecs.t1.small 。

```
EcsInstance:
  Type: ALIYUN::ECS::Instance
  InstanceType:
    Ref: EcsType
```

### Mappings（可选）
一个 Key-Value 映射表。在模板的 Resources 和 Outputs 部分可以使用 Fn::FindInMap 内部函数，通过给出 Key 获取映射表的 Value。

### 示例
```
ROSTemplateFormatVersion: '2015-09-01'
Parameters:
  regionParam:
    Description: 选择创建ECS的区域
    Type: String
    AllowedValues:
      - hangzhou
      - beijing
Mappings:
  RegionMap:
    hangzhou:
      '32': m-25l0rcfjo
      '64': m-25l0rcfj1
    beijing:
      '32': m-25l0rcfj2
      '64': m-25l0rcfj3
Resources:
  WebServer:
    Type: 'ALIYUN::ECS::Instance'
    Properties:
     # "ImageId" : { "Fn::FindInMap" : [ "RegionMap", { "Ref" : "regionParam" }, "32"]},
      ImageId:
        'Fn::FindInMap':
          - RegionMap
          - Ref: regionParam
          - '32'
      InstanceType: ecs.t1.small
      SecurityGroupId: sg-25zwc3se0
      ZoneId: cn-beijing-b
      Tags:
        - Key: tiantt
          Value: ros
        - Key: tiantt1
          Value: ros1
```

### Resources（可选）
描述堆栈中每一个资源的属性和依赖关系。一个资源可以被其他资源和 Output 所引用。

#### 语法
```
Resources:
  ID:
    Type:
      # 资源类型
    Properties:
      # 资源属性
    DependsOn: '依赖的服务的标识'
    DeletionPolicy: 'Retain'
```
- ID：自己为资源起的名字，必须是唯一的。在模板的其它地方可以使用资源标识引用这个资源
- Type，资源类型：说明一下资源的类型。比如一个阿里云 ECS 服务器的资源，它的类型应该是 `ALIYUN::ECS::Instance`，详细列表：https://ros.console.aliyun.com/#/resourceType/list
- Properties，资源属性：不同的资源类型可以配置的属性不一样，具体可用的属性要参考资源类型的详细介绍页面。阿里云 ECS 资源可用的资源属性：https://ros.console.aliyun.com/#/resourceType/detail/ALIYUN::ECS::Instance/metadata
- DeletionPolicy，删除策略：它决定了在删除堆栈时，是否要删除这个资源。想要保留可以这样：`DeletionPolicy: 'Retain'`
- DependsOn，依赖的资源：设置资源依赖的其它资源，这样会先去创建它依赖的资源。比如：`DependsOn: 'DatabseServer'`，意思是这个资源依赖一个名字是 `DatabseServer` 的资源

### Outputs（可选）
在 Outputs 部分定义在调用查询堆栈接口时返回的值。例如，用户可以定义 ECS 实例 ID 的输出，然后调用查询堆栈的接口查看该实例 ID。

#### 语法
```
Outputs:
  ID:
    Description: '描述'
    Value: '输出的值'
```
- ID: 输出的标识符
- Description（可选）：用于描述输出值的 String 类型
- Value（必需）：在调用查询堆栈接口时返回的属性值。

#### 示例
第一个输出资源 ID 为 WebServer 的 InstanceId 属性，第二个输出资源 ID 为 WebServer 的 PublicIp 属性。
```
Outputs:
  InstanceId:
    Value:
      'Fn::GetAtt':
        - 'WebServer'
        - 'InstanceId'
  PublicIp:
    Value:
     'Fn::GetAtt':
      - 'WebServer'
      - 'PublicIp'
```
### 内部函数
编排服务提供多个内置函数帮助您管理您的堆栈。可以在资源和输出中使用内部函数。

#### Fn::Base64
返回输入字符串的 Base64 编码结果。

```
Fn::Base64: stringToEncode
```

#### Fn::FindInMap
返回与 Mappings 部分声明的双层映射中的键对应的值。返回值是分配给 SecondLevelKey 的值。

```
"Fn::FindInMap" : [ "MapName", "TopLevelKey", "SecondLevelKey"]
```

- Mappings：Mappings 部分中所声明映射的 ID，包含键和值。
- TopLevelKey：第一级键，其值是一个键/值对列表。
- SecondLevelKey：第二级键，其值是一个字符串或者数字。

```
ROSTemplateFormatVersion: '2015-09-01'
Parameters:
  regionParam:
    Description: 选择创建ECS的区域
    Type: String
    AllowedValues:
      - hangzhou
      - beijing
Mappings:
  RegionMap:
    hangzhou:
      '32': m-25l0rcfjo
      '64': m-25l0rcfj1
    beijing:
      '32': m-25l0rcfj2
      '64': m-25l0rcfj3
Resources:
  WebServer:
    Type: 'ALIYUN::ECS::Instance'
    Properties:
      ImageId:
        'Fn::FindInMap':
          - RegionMap
          - Ref: regionParam
          - '32'
      InstanceType: ecs.t1.small
      SecurityGroupId: sg-25zwc3se0
      ZoneId: cn-beijing-b
      Tags:
        - Key: key1
          Value: value1
        - Key: key2
          Value: value2

```

#### Fn::GetAtt
返回模板中的资源的属性的值。

```
"Fn::GetAtt": [ "resourceID", "attributeName" ]
```

- resourceID：目标资源的 ID
- attributeName：目标资源的属性名称

示例

此示例返回 Resource ID 为 MyEcsInstance 的　ImageId 属性。

```
"Fn::GetAtt" : [ "MyEcsInstance" , "ImageID" ]
```

#### Fn::Join
把一组值连接起来，用特定分隔符隔开。

```
"Fn::Join" : [ "delimiter", [ "string1", "string2", ... ]
```

- delimiter：分隔符。分隔符可以为空，这样就将所有的值直接连接起来
- [ "string1", "string2", ... ]：被连接起来的值列表

```
"Fn::Join" : [ ",", [ "a", "b", "c" ] ]
```
返回：

"a,b,c"

支持：
- Fn::Base64
- Fn::GetAtt
- Fn::Join
- Fn::Select
- Ref

### Fn::Select
通过索引返回数据元列表中的单个数据元。
数据元列表可以是一个数组：
```
"Fn::Select" : [ "index", [ "value1", "value2", ... ] ]
```
数据元列表也可以是一个映射表：
```
"Fn::Select" : [ "index", { "key1": "value1", ... } ]
```
- index：待检索数据元的索引。如果数据元列表是一个数组，则索引是零到 N-1 之间的某个值，其中 N 代表阵列中元素的数量。如果数据元列表是一个映射表，则索引是映射表中的某个键。如果根据找不到索引对应的值，则返回空字符串。

```
{ "Fn::Select" : [ "1", [ "apples", "grapes", "oranges", "mangoes" ] ] }
```
返回："grapes"。

```
{ "Fn::Select" : [ "key1", [ "key1": "grapes", "key2": "mangoes" ] ] }
```
返回："grapes"。

#### Ref
返回指定 参数 或 资源 的值。如果指定参数是 Resource ID ，则返回资源的值。否则认为指定参数是参数，将尝试返回参数的值。

```
"Ref" : "logicalName"
```

- logicalName：您想引用的资源或参数之逻辑名称。
