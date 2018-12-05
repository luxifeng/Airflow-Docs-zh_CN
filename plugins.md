# 插件

Airflow内建一个简单的插件管理器，能够将外部特性整合到它的核心，只需简单地将文件放到你的`$AIRFLOW_HOME/plugins`文件夹。

`plugins`文件夹下的python模块会被导入，**hook**、**operator**、**sensor**、**macros**、**executor**和**web view**都会被整合到Airflow的主要集合，变成可用。

### 为何？

Airflow为数据处理提供了通用的工具箱。不同的组织有不同的技术栈和不同的需求。对于企业，使用Airflow插件是定制它们的Airflow安装来反映它们的生态系统的一种方式。

插件易于编写、共享和激活一系列特性。

同时，一些更为复杂的应用需要与不同形式的数据和元数据交互。

示例：

* 一组解析Hive日志和显示Hive元数据（CPU/IO/phases/skew/...）的工具
* 一个异常检测框架，允许用户收集指标、设置阈值、报警
* 一个审计工具，帮助理解谁能访问什么
* 一个配置驱动的SLA监控工具，允许你设置监控表和何时应着陆、警告用户、可视化地显示断电情况
* ……

### 为何构建在Airflow之上？

Airflow有很多组件，在建立应用时能被重用：

* 可用于渲染视图的web服务器
* 可用于保存模型的原数据库
* 可访问数据库，提供如何连接它们的知识
* 可承担应用负载的一组工作节点
* Airflow已部署，你可以仅仅依赖其部署逻辑
* 基本的图表功能、底层库和抽象

#### 接口

为创建一个插件，你将需要派生`airflow.plugins_manager.AirflowPlugin`类和引用你想接入Airflow的对象。你需要派生的类看起来像：

```python
class AirflowPlugin(object):
    # The name of your plugin (str)
    name = None
    # A list of class(es) derived from BaseOperator
    operators = []
    # A list of class(es) derived from BaseSensorOperator
    sensors = []
    # A list of class(es) derived from BaseHook
    hooks = []
    # A list of class(es) derived from BaseExecutor
    executors = []
    # A list of references to inject into the macros namespace
    macros = []
    # A list of objects created from a class derived
    # from flask_admin.BaseView
    admin_views = []
    # A list of Blueprint object created from flask.Blueprint. For use with the flask_admin based GUI
    flask_blueprints = []
    # A list of menu links (flask_admin.base.MenuLink). For use with the flask_admin based GUI
    menu_links = []
    # A list of dictionaries containing FlaskAppBuilder BaseView object and some metadata. See example below
    appbuilder_views = []
    # A list of dictionaries containing FlaskAppBuilder BaseView object and some metadata. See example below
    appbuilder_menu_items = []
```

#### 示例

下面的代码定义了一个插件，注入了一组虚拟对象定义到Airflow。

```python
# This is the class you derive to create a plugin
from airflow.plugins_manager import AirflowPlugin

from flask import Blueprint
from flask_admin import BaseView, expose
from flask_admin.base import MenuLink

# Importing base classes that we need to derive
from airflow.hooks.base_hook import BaseHook
from airflow.models import BaseOperator
from airflow.sensors.base_sensor_operator import BaseSensorOperator
from airflow.executors.base_executor import BaseExecutor

# Will show up under airflow.hooks.test_plugin.PluginHook
class PluginHook(BaseHook):
    pass

# Will show up under airflow.operators.test_plugin.PluginOperator
class PluginOperator(BaseOperator):
    pass

# Will show up under airflow.sensors.test_plugin.PluginSensorOperator
class PluginSensorOperator(BaseSensorOperator):
    pass

# Will show up under airflow.executors.test_plugin.PluginExecutor
class PluginExecutor(BaseExecutor):
    pass

# Will show up under airflow.macros.test_plugin.plugin_macro
def plugin_macro():
    pass

# Creating a flask admin BaseView
class TestView(BaseView):
    @expose('/')
    def test(self):
        # in this example, put your test_plugin/test.html template at airflow/plugins/templates/test_plugin/test.html
        return self.render("test_plugin/test.html", content="Hello galaxy!")
v = TestView(category="Test Plugin", name="Test View")

# Creating a flask blueprint to integrate the templates and static folder
bp = Blueprint(
    "test_plugin", __name__,
    template_folder='templates', # registers airflow/plugins/templates as a Jinja template folder
    static_folder='static',
    static_url_path='/static/test_plugin')

ml = MenuLink(
    category='Test Plugin',
    name='Test Menu Link',
    url='https://airflow.incubator.apache.org/')

# Creating a flask appbuilder BaseView
class TestAppBuilderBaseView(AppBuilderBaseView):
    @expose("/")
    def test(self):
        return self.render("test_plugin/test.html", content="Hello galaxy!")
v_appbuilder_view = TestAppBuilderBaseView()
v_appbuilder_package = {"name": "Test View",
                        "category": "Test Plugin",
                        "view": v_appbuilder_view}

# Creating a flask appbuilder Menu Item
appbuilder_mitem = {"name": "Google",
                    "category": "Search",
                    "category_icon": "fa-th",
                    "href": "https://www.google.com"}

# Defining the plugin class
class AirflowTestPlugin(AirflowPlugin):
    name = "test_plugin"
    operators = [PluginOperator]
    sensors = [PluginSensorOperator]
    hooks = [PluginHook]
    executors = [PluginExecutor]
    macros = [plugin_macro]
    admin_views = [v]
    flask_blueprints = [bp]
    menu_links = [ml]
    appbuilder_views = [v_appbuilder_package]
    appbuilder_menu_items = [appbuilder_mitem]
```

### 关于基于角色的视图

Airflow 1.10通过FlaskAppBuilder引入了基于角色的视图。你可以通过设置rbac = True配置使用哪个界面。为了支持所有版本UI的插件视图和链接，以及维护向后兼容性，appbuilder\_views和appbuilder\_menu\_items字段已被加入AirflowTestPlugin类。

