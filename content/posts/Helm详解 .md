---
title: "Helm解析"
date: 2022-07-17
categories:
- k8s
- docker
thumbnailImagePosition: right
thumbnailImage: img/main/th-13.jpeg
---

Helm概念
- Chart 是一个Helm包，涵盖了需要在Kubernetes集群中运行应用，工具或者服务的资源定义。 把它想象成Kubernetes对应的Homebrew公式，Apt dpkg，或者是Yum RPM文件。
- 仓库（Repository）： 归集和分享chart的地方。
- 发布（Release）：在Kubernetes集群中运行的chart实例。一个chart经常在同一个集群中被重复安装。每次安装都会生成新的发布。比如MySQL，如果想让两个数据库运行在集群中，可以将chart安装两次。每一个都会有自己的发布版本，并有自己的发布名称。

<!--more-->

{{< toc >}}

# Helm 安装
- Download your desired version(wget https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz)
- Unpack it (tar -zxvf helm-v3.9.0-linux-amd64.tar.gz)
- Find the helm binary in the unpacked directory, and move it to its desireddestination (mv linux-amd64/helm /usr/local/bin/helm)

# Helm 仓库

可以用helm repo list来查看当前的仓库配置
```shell
$ helm repo list
NAME       URL
stable     https://kubernetes-charts.storage.googleapis.com/
local      http://127.0.0.1:8879/charts
```
# 查找 chart
Helm 将 Charts 包安装到 Kubernetes 集群中，一个安装实例就是一个新的 Release，要找到新的 Chart，我们可以通过搜索命令完成。

```shell
# helm search 
Usage:
  helm search [command]

Available Commands:
  hub         search for charts in the Helm Hub or an instance of Monocular
  repo        search repositories for a keyword in charts


# helm search repo stable
NAME                                    CHART VERSION   APP VERSION             DESCRIPTION                                       
stable/acs-engine-autoscaler            2.2.2           2.1.1                   DEPRECATED Scales worker nodes within agent pools 
stable/aerospike                        0.3.3           v4.5.0.5                A Helm chart for Aerospike in Kubernetes          
...
```

# 安装 chart
要安装新的软件包，直接使用 helm install 命令即可。
```shell
# helm install mydb stable/mysql 
NAME: mydb
LAST DEPLOYED: Sat Oct 10 15:33:41 2020
NAMESPACE: default
...
```

# 自定义 chart

一个 chart 包就是一个文件夹的集合，文件夹名称就是 chart 包的名称，比如创建一个 mychart 的 chart 包：
```shell
# helm create mychart
Creating mychart

# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files
```
这里我们再来仔细看看 templates 目录下面的文件:
- NOTES.txt：chart 的 “帮助文本”。这会在用户运行 helm install 时显示给用户。
- deployment.yaml：创建 Kubernetes deployment 的基本 manifest
- service.yaml：为 deployment 创建 service 的基本 manifest
- ingress.yaml: 创建 ingress 对象的资源清单文件
- _helpers.tpl：放置模板助手的地方，可以在整个 chart 中重复使用


这里我们明白每一个文件是干嘛的就行，然后我们把 templates 目录下面所有文件全部删除掉，这里我们自己来创建模板文件：
```shell
# rm -rf mychart/templates/*.*
```
## 创建模板
这里我们来创建一个非常简单的模板 ConfigMap，在 templates 目录下面新建一个configmap.yaml文件
```shell
# cat mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

实际上现在我们就有一个可安装的 chart 包了，通过helm install命令来进行安装：
```shell
# helm install cmtest ./mychart/
NAME: cmtest
LAST DEPLOYED: Sat Oct 10 15:58:21 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
通过命令，我们可以看到cm已经创建
```shell
# kubectl get cm mychart-configmap
NAME                DATA   AGE
mychart-configmap   1      51s
```

上面我们定义的 ConfigMap 的名字是固定的，但往往这并不是一种很好的做法，我们可以通过插入 release 的名称来生成资源的名称，比如这里 ConfigMap 的名称我们希望是：ringed-lynx-configmap，这就需要用到 Chart 的模板定义方法了。现在我们来重新定义下上面的 configmap.yaml 文件：
```shell
# cat mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```
现在我们来重新安装我们的 Chart 包，注意观察 ConfigMap 资源对象的名称：
```shell
# helm install cmtest ./mychart
# kubectl get cm | grep configmap
cmtest-configmap   1      27s

```
可以看到现在生成的名称变成了cmtest-configmap.

## 内置对象
- Release：这个对象描述了 release 本身。它里面有几个对象：
	- Release.Name：release 名称
	- Release.Time：release 的时间
	- Release.Namespace：release 的 namespace（如果清单未覆盖）
	- Release.Revision：此 release 的修订版本号，从1开始累加。
	- Release.IsUpgrade：如果当前操作是升级或回滚，则将其设置为 true。
	- Release.IsInstall：如果当前操作是安装，则设置为 true。
- Values：从values.yaml文件和用户提供的文件传入模板的值。默认情况下，Values 是空的。
- Chart：Chart.yaml文件的内容。所有的 Chart 对象都将从该文件中获取。chart 指南中Charts Guide列出了可用字段，可以前往查看。
- Files：这提供对 chart 中所有非特殊文件的访问。虽然无法使用它来访问模板，但可以使用它来访问 chart 中的其他文件。请参阅 “访问文件” 部分。
	- Files.Get 是一个按名称获取文件的函数（.Files.Get config.ini）
	- Files.GetBytes 是将文件内容作为字节数组而不是字符串获取的函数。这对于像图片这样的东西很有用。
- Capabilities：这提供了关于 Kubernetes 集群支持的功能的信息。
	- Capabilities.APIVersions 是一组版本信息。
	- Capabilities.APIVersions.Has $version 指示是否在群集上启用版本（batch/v1）。
	- Capabilities.KubeVersion 提供了查找 Kubernetes 版本的方法。它具有以下值：Major，Minor，GitVersion，GitCommit，GitTreeState，BuildDate，GoVersion，Compiler，和 Platform。
	- Capabilities.TillerVersion 提供了查找 Tiller 版本的方法。它具有以下值：SemVer，GitCommit，和 GitTreeState。
- Template：包含有关正在执行的当前模板的信息
- Name：到当前模板的文件路径（例如 mychart/templates/mytemplate.yaml）
- BasePath：当前 chart 模板目录的路径（例如 mychart/templates）。
上面这些值可用于任何顶级模板，要注意内置值始终以大写字母开头。这也符合Go的命名约定。当你创建自己的名字时，你可以自由地使用适合你的团队的惯例。

## values 文件
Values 对象的值有4个来源：
- chart 包中的 values.yaml 文件
- 父 chart 包的 values.yaml 文件
- 通过 helm install 或者 helm upgrade 的-f 或者--values参数传入的自定义的 yaml 文件
- 通过--set 参数传入的值
chart 的 values.yaml 提供的值可以被用户提供的 values 文件覆盖，而该文件同样可以被--set提供的参数所覆盖。  

比如（values.yaml）：
```
course:  
	k8s: devops
```
使用方式(configmap.yaml)：
```
data:  
	myvalue: "Hello World"  
	course: {{ .Values.course.k8s }}
```

## Helm 模板之模板函数与管道

### 模板函数
```
# cat values.yaml 
course:
  k8s: devops
  python: django
```
- 从.Values中读取的值变成字符串的时候就可以通过调用quote模板函数来实现(templates/configmap.yaml)：
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ quote .Values.course.k8s }}
  python: {{ .Values.course.python }}
```
使用quote前后结构对比，结果加上了双引号””：
```
# 使用前
data:
  myvalue: "Hello World"
  k8s: devops
  python: django

# 使用后
data:
  myvalue: "Hello World"
  k8s: "devops"
  python: django
```

### 管道

和UNIX中一样，管道我们通常称为Pipeline，是一个链在一起的一系列模板命令的工具，以紧凑地表达一系列转换。简单来说，管道是可以按顺序完成一系列事情的一种方法。比如我们用管道来重写上面的 ConfigMap 模板（templates/configmap.yaml）：
```
k8s: {{ .Values.course.k8s | quote }}
```
k8s 的 value 值被渲染后是大写的字符串，则我们就可以使用管道来修改：（templates/configmap.yaml）
```
k8s: {{ .Values.course.k8s | upper | quote }}
```
这里我们在管道中增加了一个upper函数，该函数同样是Sprig 模板库提供的，表示将字符串每一个字母都变成大写。然后我们用debug模式来查看下上面的模板最终渲染的结果。
data:  k8s: "DEVOPS"
```
data:
  k8s: "DEVOPS"
```

使用管道操作的时候，前面的操作结果会作为参数传递给后面的模板函数，比如我们这里希望将上面模板中的 python 的值渲染为重复出现3次的字符串，则我们就可以使用到Sprig 模板库提供的repeat函数，不过该函数需要传入一个参数repeat COUNT STRING表示重复的次数：（templates/configmap.yaml）
```
python: {{ .Values.course.python | repeat 3 | quote }}
```
输出结果为
```
data:
  python: "djangodjangodjango"
```
default 函数：default DEFAULT_VALUE GIVEN_VALUE。该函数允许我们在模板内部指定默认值，以防止该值被忽略掉了。比如我们来修改上面的 ConfigMap 的模板：（templates/configmap.yaml）
```
myvalue: {{ .Values.hello | default  "Hello World" | quote }}
```
由于我们的values.yaml文件中只定义了 course 结构的信息，并没有定义 hello 的值，所以使用默认值：Hello World
```
data:
  myvalue: "Hello World"
```

## Helm 模板之控制流程

控制流程为我们提供了控制模板生成流程的一种能力，Helm 的模板语言提供了以下几种流程控制：
- if/else 条件块
- with 指定范围
- range 循环块  

除此之外，它还提供了一些声明和使用命名模板段的操作：
- define在模板中声明一个新的命名模板
- template导入一个命名模板
- block声明了一种特殊的可填写的模板区域

### if/else 条件
if/else块是用于在模板中有条件地包含文本块的方法，条件块的基本结构如下：
```
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```
当然要使用条件块就得判断条件是否为真，如果值为下面的几种情况，则管道的结果为 false：
- 一个布尔类型的假
- 一个数字零
- 一个空的字符串
- 一个nil（空或null）
- 一个空的集合（map、slice、tuple、dict、array）  

除了上面的这些情况外，其他所有条件都为真。
还是以上面的 ConfigMap 模板文件为例，添加一个简单的条件判断，如果 python 被设置为 django，则添加一个web: true：（tempaltes/configmap.yaml）

```
{{ if eq .Values.course.python "django" }} web: true {{ end }}
```

其中运算符eq是判断是否相等的操作，除此之外，还有ne、lt、gt、and、or等运算符都是 Helm 模板已经实现了的，直接使用即可。

### 空格控制
执行配置文件结果，则会有空行
```
{{ if eq .Values.course.python "django" }}
web: true
{{ end }}
```
结果示例：
```
data:
  myvalue: "Hello World"
  k8s: devops
  python: django
  
  web: true
```
我们可以看到渲染出来会有多余的空行，这是因为当模板引擎运行时，它将一些值渲染过后，之前的指令被删除，但它之前所占的位置完全按原样保留剩余的空白了，所以就出现了多余的空行。YAML文件中的空格是非常严格的，所以对于空格的管理非常重要，一不小心就会导致你的YAML文件格式错误。  

我们可以通过使用在模板标识 { 后面添加破折号和空格 前面添加一个空格和破折号 -}} 表示应该删除右边的空格，另外需要注意的是换行符也是空格！
使用这个语法，我们来修改我们上面的模板文件去掉多余的空格：（templates/configmap.yaml）
```
{{- if eq .Values.course.python "django" }}
web: true
{{- end }}
```

### 使用 with 修改范围
```
{{ with PIPELINE }}
  #  restricted scope
{{ end }}
```
with语句可以允许将当前范围.设置为特定的对象，比如我们前面一直使用的.Values.course，我们可以使用with来将.范围指向.Values.course：(templates/configmap.yaml)
```
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 3 | quote }}
  {{- if eq .python "django" }}
  web: true
  {{- end }}
  {{- end }}
```
可以看到上面我们增加了一个xxx的一个块，这样的话我们就可以在当前的块里面直接引用.python和.k8s了，而不需要进行限定了，这是因为该with声明将.指向了.Values.course，在后.就会复原其之前的作用范围了，我们可以使用模板引擎来渲染上面的模板查看是否符合预期结果。  

不过需要注意的是在with声明的范围内，此时将无法从父范围访问到其他对象了，比如下面的模板渲染的时候将会报错，因为显然.Release根本就不在当前的.范围内，当然如果我们最后两行交换下位置就正常了，因为之后范围就被重置了：
```
{{- with .Values.course }}
k8s: {{ .k8s | upper | quote }}
python: {{ .python | repeat 3 | quote }}
release: {{ .Release.Name }}
{{- end }}
```

### range 循环
在 Helm 模板语言中，是使用range关键字来进行循环操作。
我们在values.yaml文件中添加上一个课程列表：
```
course:
  k8s: devops
  python: django
courselist:
- k8s
- python
- search
- golang
```
修改 ConfigMap 模板文件来循环打印出该列表：
```
toppings: |-
  {{- range .Values.courselist }}
  - {{ . | title | quote }}
  {{- end }}
```

可以看到最下面我们使用了一个range函数，该函数将会遍历 列表，循环内部我们使用的是一个.，这是因为当前的作用域就在当前循环内，这个.从列表的第一个元素一直遍历到最后一个元素，然后在遍历过程中使用了title和quote这两个函数，前面这个函数是将字符串首字母变成大写，后面就是加上双引号变成字符串，所以按照上面这个模板被渲染过后的结果为：
```
toppings: |-
  - "Mushrooms"
  - "Cheese"
  - "Peppers"
  - "Onions"
```

我们可以看到courselist按照我们的要求循环出来了。除了 list 或者 tuple，range 还可以用于遍历具有键和值的集合（如map 或 dict），这个就需要用到变量的概念了。

### 变量

在 Helm 模板中，使用变量的场合不是特别多，但是在合适的时候使用变量可以很好的解决我们的问题。如下面的模板：
```
{{- with .Values.course }}
k8s: {{ .k8s | upper | quote }}
python: {{ .python | repeat 3 | quote }}
release: {{ .Release.Name }}
{{- end }}
```
我们在with语句块内添加了一个.Release.Name对象，但这个模板是错误的，编译的时候会失败，这是因为.Release.Name不在该with语句块限制的作用范围之内，我们可以将该对象赋值给一个变量可以来解决这个问题：
```
data:
  {{- $releaseName := .Release.Name -}}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 3 | quote }}
  release: {{ $releaseName }}
  {{- end }}
```

我们可以看到我们在with语句上面增加了一句 ，其中$releaseName就是后面的对象的一个引用变量，它的形式就是$name，赋值操作使用:=，这样with语句块内部的$releaseName变量仍然指向的是.Release.Name。  

另外变量在range循环中也非常有用，我们可以在循环中用变量来同时捕获索引的值：
```
toppings: |-
  {{- range $index, $course := .Values.courselist }}
  - {{ $index }}: {{ $course | title | quote }}
  {{- end }}
```

## Helm模板之命名模板

### 声明和使用命名模板
使用define关键字就可以允许我们在模板文件内部创建一个命名模板，它的语法格式如下：
```
{{ define "ChartName.TplName" }}
# 模板内容区域
{{ end }}
```
然后我们可以将该模板嵌入到现有的 ConfigMap 中，然后使用template关键字在需要的地方包含进来即可：
```
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```
我们可以看到define区域定义的命名模板被嵌入到了template所在的区域，但是如果我们将命名模板全都写入到一个模板文件中的话无疑也会增大模板的复杂性。   

还记得我们在创建 chart 包的时候，templates 目录下面默认会生成一个_helpers.tpl文件吗？我们前面也提到过 templates 目录下面除了NOTES.txt文件和以下划线_开头命令的文件之外，都会被当做 kubernetes 的资源清单文件，而这个下划线开头的文件不会被当做资源清单外，还可以被其他 chart 模板中调用，这个就是 Helm 中的partials文件，所以其实我们完全就可以将命名模板定义在这些partials文件中，默认就是_helpers.tpl文件了。  

现在我们将上面定义的命名模板移动到 templates/_helpers.tpl 文件中去：
```
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
```
一般情况下面，我们都会在命名模板头部加一个简单的文档块，用/**/包裹起来，用来描述我们这个命名模板的用途的。  

现在我们讲命名模板从模板文件 templates/configmap.yaml 中移除，当然还是需要保留 template 来嵌入命名模板内容，名称还是之前的 mychart.lables，这是因为模板名称是全局的，所以我们可以能够直接获取到。

### 模板范围
上面我们定义的命名模板中，没有使用任何对象，只是使用了一个简单的函数，如果我们在里面来使用 chart 对象相关信息呢：
```
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```
如果这样的直接进行渲染测试的话，是不会得到我们的预期结果的,直接报错：
```
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: [unknown object type "nil" in ConfigMap.metadata.labels.chart, unknown object type "nil" in ConfigMap.metadata.labels.version]
```
ConfigMap.metadata.labels.chart 的值不对。 （正好是我们yaml文件的层级结构）  

chart 的名称和版本都没有正确被渲染，这是因为他们不在我们定义的模板范围内，当命名模板被渲染时，它会接收由 template 调用时传入的作用域，由于我们这里并没有传入对应的作用域，因此模板中我们无法调用到 .Chart 对象，要解决也非常简单，我们只需要在 template 后面加上作用域范围即可：

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```

我们在 template 末尾传递了.，表示当前的最顶层的作用范围，如果我们想要在命名模板中使用.Values范围内的数据，当然也是可以的。

### include 函数

假如现在我们将上面的定义的 labels 单独提取出来放置到 _helpers.tpl 文件中：
```
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
from: helm
date: {{ now | htmlDate }}
chart: {{ .Chart.Name }}
version: {{ .Chart.Version }}
{{- end }}
```

现在我们将该命名模板插入到 configmap 模板文件的 labels 部分和 data 部分：

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
    {{- template "mychart.labels" . }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
  {{- template "mychart.labels" . }}
```

我们可以看到渲染结果是有问题的，不是一个正常的 YAML 文件格式，这是因为template只是表示一个嵌入动作而已，不是一个函数，所以原本命名模板中是怎样的格式就是怎样的格式被嵌入进来了，比如我们可以在命名模板中给内容区域都空了两个空格，再来查看下渲染的结构：  

为了解决这个问题，Helm 提供了另外一个方案来代替template，那就是使用include函数，在需要控制空格的地方使用indent管道函数来自己控制，比如上面的例子我们替换成include函数：
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
{{- include "mychart.labels" . | indent 4 }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- include "mychart.labels" . | indent 2 }}
```

可以看到是符合我们的预期，所以在 Helm 模板中我们使用 include 函数要比 template 更好，可以更好地处理 YAML 文件输出格式。