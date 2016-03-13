# 第二章 Building Your First Odoo Application
************

Developing in Odoo most of the time means creating our own modules. In this
chapter, we will create our first Odoo application, and you will learn the steps
needed make it available to Odoo and install it.  

在Odoo中开发，大多是时间意味着创建我们自己的模块。本章，我们将创建自己的第一个Odoo应用，你也会学习到使应用在Odoo中生效的必要步骤，并对应用进行安装。  

Inspired by the notable todomvc.com project, we will build a simple to-do application. It should allow us to add new tasks, then mark them as completed, and finally clear the task list of all completed tasks.  

受到著名的todomvc.com项目启发，我们将构建一个简单的计划清单应用。它应该能让我们添加新任务，然后对任务标记完成，最后可以清楚任务清单上的所有已完成任务。  

You will learn how Odoo follows an MVC architecture, and we will go through the
following layers during the to-do application implementation:  

你将学习的到Odoo是如何遵循了MVC架构，在todo应用实现时我们会将下面的三个层贯穿到其中：  

- The model, defining the structure of the data
- The view, describing the user interface
- The controller, supporting the business logic of the application

- 模型，定义数据结构
- 视图，描述用户接口
- 控制器，对应用的业务逻辑提供支持

The model layer is defined with Python objects that have their data is stored in the
PostgreSQL database. The database mapping is automatically managed by Odoo, and the mechanism responsible for this is the object relational model, (ORM).  

模型层使用Python对象定义，而模型的数据存储在PostgreSQL数据库中。数据库映射由Odoo来自动地管理，负责管理的映射机制称作“对象关系模型（ORM）”。  

The view layer describes the user interface. Views are defined using XML, which is
used by the web client framework to generate data-aware HTML views.  

视图层描述用户接口。视图使用XML定义，它被客户端框架用来生成人类可感知的HTML视图。  

The web client views perform data persistent actions by interacting with the server ORM. These can be basic operations such as write or delete, but can also invoke methods de ned in the ORM Python objects, performing more complex business logic. This is what we refer to as the controller layer.  

Web客户端视图执行数据保存的动作是通过与服务器ORM交互来完成的。这些基本操作可以是写或者删，但是你也可以调用定义在ORM 对象中的方法，以便执行更为复杂的业务逻辑。这就是我们将说到的控制器层。  

>### Note
>Note that the concept of controller mentioned here is different from the Odoo web development controllers. Those are program endpoints that web pages can call to perform actions.  

>### 注释
>注意这里提到的控制器概念不同于Odoo Web开发中的控制器。这些控制器是可以被web页面调用来执行动作的程序端点。  

With this approach, you will be able to gradually learn about the basic building blocks that make up an application and experience the iterative process of building an Odoo module from scratch.  

使用该方法，你能够逐步的学习可以让程序运行起来的代码段，也会体验到从零开始构建Odoo模块的交互过程。  
 
## Understanding applications and modules 理解应用和模块
It's common to hear about Odoo modules and applications. But what exactly is the difference between them? Modules are building blocks of Odoo applications. A module can add or modify Odoo features. It is supported by a directory containing a manifest or descriptor file (named `__openerp__.py`) and the remaining files that implement its features. Sometimes, modules can also be referred to as "add-ons." Applications are not different from regular modules, but functionally, they provide a central feature, around which other modules add features or options. They provide the core elements for a functional area, such as accounting or HR, around which other modules add features. Because of this, they are highlighted in the Odoo Apps menu.  

Odoo模块和应用经常被提及，但是它们之间到底有什么区别呢？模块用来构建Odoo应用的区块。模块可以添加或者修改Odoo的功能。通过在目录中包含一个明显的描述符文件（称作`__openerp__.py`）,以及剩余的文件实现模块功能。有时候，模块可以称作“附加插件”。应用不同于常规的模块，从功能上来说，应用提供了核心功能，并为其他模块提供了功能添加和一些选项。应用对功能区提供了核心元素，围绕着其他模块的功能添加，比如账单或者人力资源。所以，应用常常在Odoo的Apps菜单中高亮显示。  

## Modifying and extending modules 模块的修改和扩展
In the example that will follow, we will create a new module with as few dependencies as possible.  

在接下来的例子中，我们将创建一个尽可能少些依赖的模块。  

This will not be the typical case, however. The most frequent situation is where
modi cations or extensions are needed on an already existing module to fit some specific use cases.  

不过，这种情况并不常见。最常见的情况是按照需求修改或者扩展一个已经存在的模块，以便适应默写特定应用场合。  

The golden rule is that we shouldn't modify existing modules by changing them directly. It's considered bad practice to modify existing modules. This is especially true for the of cial modules provided by Odoo. Doing so does not allow a clear separation between the original module code and our modi cations, and makes it dif cult to apply upgrades.  

黄金定律是我们不能直接地改变已存在的模块。修改已存在的模块被认为是糟糕的做法。这个法则对于Odoo官方提供的模块更是如此。在原生模块代码和我们的自定义的代码之间这样操作是不允许明确分离的，同时也会让升级变得很难。  

Instead, we should create new modules to be applied on top of the modules we want to modify, and implement those changes. This is one of Odoo's main strengths: it provides "inheritance" mechanisms that allow custom modules to extend existing modules, either of cial or from the community. The inheritance is possible at all levels data models, business logic, and user interface layers.  

与之相反，我们应该在我们想要修改的模块顶层创建新的模块，再去执行这些变更。这是Odoo的主要的力量体现之一：它的提供了允许用户自定义模块，以扩展来自官方或者社区的模块的“继承”机制。这种

Right now, we will create a completely new module, without extending any existing module, to focus on the different parts and steps involved in module creation. We will just take a brief look at each part, since each will be studied in more detail in the later chapters. Once we are comfortable with creating a new module, we can dive into the inheritance mechanisms, which will be introduced in the next chapter.  

现在，我们要创建一个完整的新模块，而不是扩展任何已存在的模块，注重模块创建中的不同部分和步骤。我们将简短的浏览每个部分，因为每个部分都会在后面的章节中进行详细学习。一旦我们适应了创建新模块，我们可以深入到会在下一章中引入的继承机制中。  

## Creating a new module 创建新模块
Our module will be a very simple application to keep to-do tasks. These tasks will have
a single text field, for the description, and a checkbox to mark them as complete. We
will also have a button to clean the to-do list from the old completed tasks.  

我们的模块是一个非常简单的保存任务清单的应用。这些任务拥有一个简单的文本字段，为了便于描述，使用复选框来标记任务的完成。我们也会拥有一个从旧的已完成任务中清除任务清单的按钮。  

These are very simple specifications, but throughout the book we will gradually add
new features to it, to make it more interesting for the users.  

指定这些内容非常简单，在本书中我们会逐步地添加一些新特性到描述符文件，以便用户对它更感兴趣。  

Enough talk, let's start coding and create our new module.  

闲话少说，然我们开始编写代码，创建自己的新模块吧。  

Following the instructions in Chapter 1, Getting Started with Odoo Development, we should have the Odoo server at /odoo-dev/odoo/. To keep things tidy, we will create a new directory alongside it to host our custom modules:  

紧接着第一章的介绍，我们开始Odoo开发，我们应该服务器应该位于/odoo-dev/odoo/。为了保持整洁，我们在这个目录旁边创建一个新的目录以托管自定义模块。  

```bash
$ mkdir ~/odoo-dev/custom-addons
```

An Odoo module is a directory containing an `__openerp__.py` descriptor file. This is still a legacy from when Odoo was named OpenERP, and in the future is expected to become `__odoo__.py`.  

Odoo模块是一个包含描述符`__openerp__.py`的目录。这名字来源自Odoo还被称作OpenERP的时候，在未来会被称作`__odoo__.py`。  

It also needs to be Python importable, so it must also have an `__init__.py` file.  

它也需要成为可导入到Python包，所以它必须拥有一个`__init__.py`文件。  

The module's directory name will be its technical name. We will use `todo_app` for it. The technical name must be a valid Python identi er: it should begin with a letter and can only contain letters, numbers, and the underscore character. The following commands create the module directory and create an empty `__init__.py`  file in it:  

模块的目录名称会成为自身技术名。我们要使用的是`todo_app`这个名字。技术名必须是一个有效的Python识别符：它应该以字母开始，而且只能包含字母、数字、和下划线。下面的命令创建了模块目录，然后在模块目录后创建了一个空文件`__init__.py`：  


```bash
$ mkdir ~/odoo-dev/custom-addons/todo_app
$ touch ~/odoo-dev/custom-addons/todo_app/__init__.py
```

Next we need to create the descriptor file. It should contain only a Python dictionary
with about a dozen possible attributes, of which only the name attribute is required. A longer description attribute and the author also have some visibility and
are advised.  

接下来，我们需要创建一个描述符文件。它应该只包含一个可能拥有一打属性的Python字典，其中只有name属性是必须的。详细的描述属性和作者也其他一些明显的属性也是我们所推荐的。  

We should now add an `__openerp__.py` file alongside the `__init__.py`  le with the following content:  

我们应该添加一个和 `__init__.py`放到一起的`__openerp__.py`文件

```json
   {
       'name': 'To-Do Application',
       'description': 'Manage your personal Tasks with this module.',
       'author': 'Daniel Reis',
       'depends': ['mail'],
       'application': True,
  }
```

The depends attribute can have a list of other modules required. Odoo will have them automatically installed when this module is installed. It's not a mandatory attribute, but it's advised to always have it. If no particular dependencies are needed, we should depend on the special base module. You should be careful to ensure all dependencies are explicitly set here, otherwise the module may fail to install in a clean database (due to missing dependencies) or have loading errors, if the other needed modules are loaded afterwards. For our application, we want to depend on the mail module because that is the module that adds the **Messaging** top menu, and we will want to include our new menu options there.  

depends属性拥有一个其它所需模块构成的列表。Odoo会在该模块安装时自动地安装依赖。这并不是必写的属性，不过这里建议你写上去。如果不需要特殊的依赖，我们应该使用特殊的base模块作为依赖。你应该用心地保证所有的依赖都是正确的，否则在清理数据库（处理缺失的依赖）时模块可能会安装失败，或者是在其他的必需模块向后载入时发生载入错误。就我们的应用而言，我们想依赖mail模块，因为，这是一个添加到消息顶层菜单中的模块，而去我们也想让自己的菜单出现在这里。  

To be concise, we chose to use very few descriptor keys, but in a real word scenario it is recommended to also use these additional keys, since they are relevant for the Odoo app store:  

为了简洁期间，我们选择使用很少的几个描述符键，但是在真实应用场景中，建议使用下面这些额外的键，因为这些键都与Odoo应用商店有关：  

- `summary` is displayed as a subtitle for the module.
- `version`, by default, is 1.0. Should follow semantic versioning rules
(see semver.org for details).
- `license` identifier, by default is AGPL-3.
- `website` is a URL to find more information about the module. This can help people to find more documentation or the issue tracker to file bugs and suggestions.
- `category` is the functional category of the module, which defaults to Uncategorized. The list of existing categories can be found in the security Groups form (`Settings` | `User` | `Groups` menu), in the **Application** field drop-down list.

- `summary` 显示模块的副标题。  
- `version` 默认是1.0. 应该遵循语意话的版本命令规则（详情见emver.org）  
- `license` 标识符号，默认为AGPL-3。  
- `website` 是有个可以找到关于模块的更多消息的URL。
- `category` 是模块的功能分类，默认是不分类的。

These other descriptor keys are also available:  

另外的一些描述符也是可以使用的：  

- installable is by default True, but can be set to False to disable a module.
- auto_install if this is set to True this module is automatically installed if all its dependencies are already installed. It is used for **glue** modules.

- `installable` 默认为True，但是可以设置为Fasle来禁用模块。
- auto_install 如果改它设置为True，那么在模块所需依赖都已经安装的情况下，该模块会自动地安装。它被当做“胶水”模块来使用。  


Since Odoo 8.0, instead of the description key we can use a README.rst or README.md  le in the module's top directory.  

从Odoo 8.0开始，我们可以在模块的顶层目录中使用README.rst 或者 README.md而不是使用描述符键。  

## Adding to the addons path 添加插件路径
Now that we have a new module, even if minimal, we want to make it available in Odoo.  

现在我们已经拥有新的模块了，尽管它不是很大，我们也想在Odoo中使用它。  

For that, we need to make sure the directory the module is in is part of the addons path. And then we need to update the Odoo module list.  
为此，我们需要保证模块的目录在插件路径的一部分。然后我们需要更新Odoo的模块列表。  

Both operations have been explained in detail in the previous chapter, but we will follow here with a brief overview of what is needed.  

这两个操作都在前面一章详细说明过了，不过这里我们要做一个需求的简要概览。  

We will position in our work directory and start the server with the appropriate
addons path configuration:  

我们从当前的工作目录中使用合适的插件路径配置来启动服务器：  

```bash
$ cd ~/odoo-dev
$ odoo/odoo.py -d v8dev --addons-path="custom-addons,odoo/addons" --save
```

The `--save` option saves the options you used in a config file. This spares you from repeating them the next time you restart the server: just run ./odoo.py and the last saved options will be used.  

--save选项会把你所使用选项保存到一个配置文件。这个副本会在下一次你启动服务器时被重复使用：运行./odoo.py，之后上次保存的选项将被使用。  

Look closely at the server log. It should have an **INFO ? openerp: addons paths**: (...) line, and it should include our custom-addons directory.  

仔细来看一看服务器日志。它应该包含**INFO ? openerp: addons paths**这样的一行，这一行也应该包含我们的custom-addons目录。  

Remember to also include any other addons directories you might be using. For instance, if you followed the last chapter's instructions to install the department repository, you might want to include it and use the option:  

不要忘了将任何你会用的目录都给包括进去。例如，你依照上一章的说明来安装部门仓储，你需要应用以下选项来包括它：  

```
--addons-path="custom-addons,departmernt,odoo/addons"
```

Now let's ask Odoo to acknowledge the new module we just added.  

现在，让我们向Odoo请求，以确认刚添加的新模块。  

For that, in the **Modules** section of the **Settings** menu, select the **Update Modules List** option. This will update the module list adding any modules added since the last update to the list. Remember that we need the Technical Features enabled for this option to be visible. That is done by selecting the **Technical Features** checkbox for our user.  

因此，在设置菜单的模块区域，请选择更新模块列表选项。该操作将更新模块列表，以添加从上次更新到列表中的已添加模块。记住，我们需要启用技术特性以便让该选项正常显示。对用户来说，你可以通过选择Technical Features（技术特性）复选框来完成这个操作。  

## Installing the new module 安装新模块
The **Local Modules** option shows us the list of available modules. By default it shows only Apps modules. Since we created an application module we don't need to remove that  lter to see it. Type "todo" in the search and you should see our new module, ready to be installed.  

本地模块选项对我们展示了可用模块列表。它默认只显示Apps模块。因为我们创建的是一个应用模块，所以我们需要移除过滤器来查看它。在搜索框内输入“todo”，你可以看到自定义的待安装新模块。  

omit:img  

Now click on its **Install** button and you're done!   

现在点击“安装”按钮，你就完成了对新模块的安装！  

## Upgrading a module 更新模块
Developing a module is an iterative process, and you will want changes made on source files to be applied and visible in Odoo.  

开发模块是一个迭代的过程，你需要去改变的源文件才能让模块在Odoo中显示出来。  

In most cases this is done by upgrading the module: look up the module in the **Local Modules** list and, since it is installed, you will see an **Upgrade** button available.  

很多情况下本操作都是通过升级模块来完成的：在本地模块列表中查看模块，因为模块已经被安装了，所以你会看到一个可用的升级按钮。  

However, when the changes are only in Python code, the upgrade may not have an effect. Instead of a module upgrade, an application server restart is needed.  

不过，当只是改变Python中的代码的话，升级是不会生效的。与升级模块相反，应用服务器的重启则是必需的。  

In some cases, if the module has changed both in data files and Python code, you
might need both operations. This is a common source of confusion for newcomer Odoo developers.  

在某些情况下，如果模块在数据文件和Python代码中都做出了改变，那么你或许要执行这两种操作。这对于Odoo开发新手来说

But fortunately, there is a better way. The simplest and fastest way to make all our changes to a module effective is to stop (Ctrl + C) and restart the server process requesting our modules to be upgraded on our work database.  

不过，幸运的是，还有更好的办法。使应用到模块的变更生效的最简单也最快的办法就是停止服务器（Ctrl + C），然后重启后服务器进程将基于当前使用的模型，来更新模块。  

To start the server upgrading the todo_app module in the v8dev database, we will use:  

为了启动服务器以更新v8dev数据库中的todo_app，我们需要用到下面的命令：  

```bash
$ ./odoo.py -d v8dev -u todo_app
```

The -u option (or --update in the long form) requires the -d option and accepts a comma-separated list of modules to update. For example, we could use:-u todo_app,mail.  

-u选项（或者非简写模式下的--update ）需要-d选项，并接受用逗号分隔的需要更新模块列表。例如，我们可以使用-u todo_app,mail。  

Whenever you need to upgrade work in progress modules throughout the book, the safest way to do so is to go to the terminal window where you have Odoo running, stop the server, and restart it with the command above. Frequently pressing the Up arrow key will be enough, since it should bring you the previous command you used to start the server.  

不论什么时候，

Unfortunately, updating the module list and uninstalling modules are both actions not available through the command line. These have to be done through the web interface, in the Settings menu.  

不幸的是，更新模块列表，以及卸载模块都是不可以通过命令行来完成的动作。这些动作必需通过web管理界面中的Settings菜单来完成。  

## Creating an application model 创建应用模型
Now that Odoo knows about our new module, let's start by adding to it a simple model.  

现在，Odoo已经知道了新的模块，让我们从为它添加一个简单的模型开始。  

Models describe business objects, such as an opportunity, a sales order, or a partner
(customer, supplier, and so on.). A model has a list of attributes and can also define its specific business.  

模型描述业务对象，比如一次机会，一个销售订单，或者业务伙伴（消费者，供应商，等等）。模型拥有一个属性列表，

Models are implemented using a Python class derived from an Odoo template class. They translate directly to database objects, and Odoo automatically takes care of that when installing or upgrading the module.  

模型是使用派生自Odoo模板类的Python类来实现的。这些模型被直接地翻译为数据库对象，当安装或者升级模块时Odoo也会自动地处理它们。  

Some consider it good practice to keep the Python files for models inside a models subdirectory. For simplicity we won't be following that here, so let's create a todo_model.py file in the todo_app module main directory.  

将模型文件放到一个models子目录内可以看作是最佳实践。不过，简单起见，我们不会按照最佳实践做，所以我们在todo_app模块主目录内创建todo_model.py文件：  

Add the following content to it:  

添加以下内容到模型文件：  

```python
   # -*- coding: utf-8 -*-
   from openerp import models, fields

   
   class TodoTask(models.Model):
       _name = 'todo.task'
       name = fields.Char('Description', required=True)
       is_done = fields.Boolean('Done?')
       active = fields.Boolean('Active?', default=True)
```

The first line is a special marker telling the Python interpreter that this file has
UTF-8, so that it can expect and handle non-ASCII characters. We won't be using any, but it's safer to use it anyway.  

第一行是一个告诉Python解释器该文件使用的是UTF－8的特殊标记，因此它可以用来处理非－ASCII字符。我们不再使用除此之外的任何东西，而且为了安全起见最好还是使用它。  

The second line makes available the models and fields objects from the Odoo core.  

第二行从Odoo内核构建可用的models和fields对象。  

The third line declares our new model. It's a class derived from models.Model. The next line sets the `_name` attribute defining the identifier that will be used throughout Odoo to refer to this model. Note that the actual Python class name is meaningless to the other Odoo modules. The `_name` value is what will be used as an identifier.  

第三行生命我们的新模型。这是一个派生自models.Model的类。接下来的行设置了定义标识符的`_name`属性，该属性在Odoo中对该模型进行全局引用。注意，实际的Python类名称对于其它Odoo模块是无意义的。`_name`的值被当做一个标识符来使用。  

Notice that this and the following lines are indented. If you're not familiar with
Python you should know that this is important: indentation defines a nested code
block, so these four line should all be equally indented.  

注意，本行以及接下来的行都是缩进的。如果你不熟悉Python，那么你应该知道这件事非常重要：缩进定义了嵌套的代码块，因此这四行都应该使用相同的缩进。  

The last three lines define the model's fields. It's worth noting that name and active are names of special fields. By default Odoo will use the name  eld as the record's title when referencing it from other models. The active field is used to inactivate records, and by default only active records will be shown. We will use it to clear away completed tasks without actually deleting them from the database.  

最后三行定义了模型的字段。特殊字段的那么。默认，当Odoo从其它模型引用该模型事，会使用名称字段作为记录的标题。active字段被用做对记录的实效进行标记，而且默认仅有活动记录才被显示。我们要用它来清除整个任务而不会把这些字段从数据库中删除。 

Right now, this file is not yet used by the module. We must tell Odoo to load it with the module in the `__init__.py` file. Let's edit it to add the following line:  

现在，这个文件还没有被模块使用。我们必需在`__init__.py`文件中告诉Odoo使用模块载入这个文件。让我们编辑这个文件，并添加下面的行：  

```python
   from . import todo_model
```

That's it. For our changes to take effect the module has to be upgraded. Locate the To-Do application in the Local Modules and click on its Upgrade button.  

大概就是这样。为了变更生效，模块必须升级。找到位于本地模块中的To－Do应用，然后点击模块的升级按钮。  

Now we can inspect the newly created model in the Technical menu. Go to Database Structure | Models and search for the todo.task model on the list. Then click on it to see its definition:   

现我们可以检查在技术菜单中新创建的模型。到“数据库结构｜模型”中去搜索列表中todo.task模型。然后点击模型，便可以看到模型的定义：  

img:omit  

If everything went right, this will let us confirm that the model and our fields were created. If you made changes and don't see them here, try a server restart, as described before, to force all of the Python code to be reloaded.  

如果没有问题的话，通过查看模型定义可以让我们确认之前创建的模型和字段。要是你改变了模型却没有在这里找到它们的话，请尝试重启服务器，就像之前所说的那样，这样做可以强制重新载入Python代码。  

We can also see some additional fields we didn't declare. These are the five reserved fields Odoo automatically adds to any model. They are as follows:  

我们可以看到我们没有声明的额外字段。它们是Odoo自动添加到任意模型的保留字段。这些字段如下：  

- id: This is the unique identifier for each record in the particular model.
- create_date and create_uid: These tell us when the record was created and who created it, respectively.
- write_date and write_uid: These tell us when the record was last modified and who modified it, respectively.

- id: 这是在特殊模型中每个记录都要使用的唯一标识符号。  
- create_date and create_uid: 这写字段分别告诉我们，记录何时被创建，由什么人创建的。  
- write_date and write_uid: 这些字段分别告诉我们字段的修改实践以及修改人。  

## Adding menu entries 添加菜单项
Now that we have a model to store our data, let's make it available on the user interface.  

现在，我们拥有了一个存储数据的模型，让我们将它显示在用户界面上吧。  

All we need to do is to add a menu option to open the To-do Task model so
that it can be used. This is done using an XML file. Just as in the case of models, some people consider it good practice to keep the view de nitions inside a views subdirectory.  

我们要做的就是添加一个菜单项以便打开To-do Task 模型，这样它才可以被使用。可以使用XML文件来实现这个目的。就像在模型例子中所做的那样，有些人认为最佳时间就是在子目录views中定义视图。  

We will create a new todo_view.xml data file in the module's top directory, and it will declare a menu item and the action performed by it:  

我们将在模块的顶层目录中创建一个新的数据文件todo_view.xml，它会声明一个菜单项，以及该菜单所执行的动作：  

```xml
   <?xml version="1.0"?>
   <openerp>
    <data>
       <!-- Action to open To-do Task list -->
       <act_window id="action_todo_task"
         name="To-do Task"
         res_model="todo.task"
         view_mode="tree,form" />
       <!-- Menu item to open To-do Task list -->
       <menuitem id="menu _todo_task"
         name="To-Do Tasks"
         parent="mail.mail_feeds"
         sequence="20"
         action="action_todo_task" />
    </data>
  </openerp>
```

The user interface, including menu options and actions, is stored in database tables.
The XML file is a data file used to load those definitions into the database when the module is installed or upgraded. This is an Odoo data file, describing two records to add to Odoo:  

用户接口，以及菜单选项和动作都存储在数据库表中。 在模块安装和升级时，XML文件用来载入这些定义到数据库的一个文件。这是一个添加了两个record到Odoo的数据文件。  

- The `<act_window>` element defines a client-side Window Action to open the todo.task model defined in the Python file, with the tree and form views enabled, in that order.  
- The `<menuitem>` defines a menu item under the Messaging menu (identified by `mail.mail_feeds`), calling the `action_todo_task` action, which was defined before. The sequence lets us set the order of the
menu options.  

- `<act_window>`定义元素定义了用户“窗口动作”，一打开定义在Python文件中的todo.task模型，同时按照顺序启用了tree和form视图。  
- `<menuitem>`定义了“消息”菜单（通过`mail.mail_feeds`识别）下面的一个菜单选项，它调用的是上面定义的`action_todo_task` 动作。sequence让我们设置菜单选项的顺序。  

Now we need to tell the module to use the new XML data file. That is done in the `__openerp__.py` file using the data attribute. It defines the list of  les to be loaded by the module. Add this attribute to the descriptor's dictionary:  

我们需要告诉模块去使用新的XML数据文件。在`__openerp__.py`写上数据属性即可完成。它定义了可以被模块载入的文件列表。添加下面的属性到描述符字典：  

```python
'data': ['todo_view.xml'],
```

Now we need to upgrade the module again for these changes to take effect. Go to the Messaging menu and you should see our new menu option available.  

现在，我们需要再次升级模块，以便应用这些变更。找到消息菜单，你应该可以看到新的菜单选项。  

img:omit  

Clicking on it will open an automatically generated form for our model, allowing us to add and edit records.  

点击它将打开一个为模型而自动生成的表单，而这个表单则允许我们添加和编辑记录。  

Views should be defined for models to be exposed to the users, but Odoo is nice
enough to do that automatically if we don't, so we can work with our model right
away, without having any form or list views defined yet.  

视图是为需要暴露给用户的模型而定义的，不过，要是我们不去做话，Odoo也会自动地来帮助我们实现视图，所以我们可以马上使用模型，而不用去定义面前还没定义的表单视图或者列表视图。  

So far, so good! Let's improve our user interface now. Try the gradual improvements as shown in the next sections, doing frequent module upgrades, and don't be afraid to experiment.  

到目前为止，还算可以！现在，我们来改进用户界面。在下一节中，我们会试着渐进式的改进，

>### Tips
>In case an upgrade fails because of an XML error, don't panic! Comment out the last edited XML portions, or remove the XML file from `__openerp__.py`, and repeat the upgrade. The server should start correctly. Now read the error message in the server log carefully—it should point you to where the problem is.  

>### 提示
>在碰到因为XML错误引起的更新失败时，不要怕！注释掉最后编辑XML的部分，或者是从`__openerp__`中溢出XML文件，然后再次执行升级。服务器应该会正常启动。现在仔细地去读服务器日志中的错误消息，日志里应该可以找到问题所在。  

## Creating views – form, tree, and search 创建视图——表单、树形、和搜索
As we have seen, if no view is defined, Odoo will automatically generate basic views to get you going. But surely you would like to define the module views yourself, so that's what we'll do next.  

正如我们所看的，如果定义了视图，Odoo为让你继续操作会自动生成基础视图。不过，你一定想要定义自己的模块视图，这也是接下来我们要做的事情。  

Odoo supports several types of views, but the three main ones are: list (also called tree), form, and search views. We'll add an example of each to our module.  

Odoo支持多种类型的视图，不过只有三中是主要的：列表（也称为树形），表单、以及搜索视图。我们为每一个模块都添加一个例子。  

All views are stored in the database, in the ir.model.view model. To add a view in a module, we declare a `<record>` element describing the view in an XML file that will be loaded into the database when the module is installed.  

所有的视图都存储在数据库里的ir.model.view模型中。要在一个模块中添加视图，我们声明了一个在模块安装时会被载入到数据库中的XML文件中的描述视图的`<record>`记录。  

### Creating a form view 创建一个表单视图
Edit the XML we just created to add this `<record>` element just after the `<data>` opening tag at the top:  

编辑刚才创建的XML，将`<record>`元素添加到在顶部的`<data>`开放标签之后：  

```xml
   <record id="view_form_todo_task" model="ir.ui.view">
     <field name="name">To-do Task Form</field>
     <field name="model">todo.task</field>
     <field name="arch" type="xml">
       <form string="To-do Task">
         <field name="name"/>
         <field name="is_done"/>
         <field name="active" readonly="1"/>
        </form>
     </field>
   </record>
```

This will add a record to the model `ir.ui.view` with the identifier `view_form_ todo_task`. The view is for the model todo.task and named To-do Task Form. The name is just for information, does not have to be unique, but should allow one to easily identify what record it refers to.  

上面的内容使用标识符`view_form_ todo_task`对模型`ir.ui.view`添加一个记录。todo.task模型的视图被称作 `To-do Task Form`。名称仅是信息而已，没有让名称唯一化的必要，但应该是可以标示出自己在引用什么记录。  

The most important attribute is arch, containing the view definition. Here we say it's a form, and it contains three fields, and we chose to make the active field read only.  

最重要的属性是定义是包含了视图定义的arch。这里我们称它为表单，它包含了三个字段，我们选择让活动字段变为只读。  

### Formatting as a business document 为业务文档进行格式化
The above provides a basic form view, but we can make some improvements to make it nicer. For document models Odoo has a presentation style that mimics a paper page. The form contains two elements: a `<head>`, containing action buttons, and a `<sheet>`, containing the data fields:  

上面提供了一个基本的表单视图，不过我们可以做出一些改进，让它显示更为友好。关于文档模型Odoo使用的是一个仿纸质页面的表现风格。表单包含了两个元素：包含动作按钮的`<head>`，以及包含数据字段的`<sheet>`：  

```xml
   <form>
     <header>
     <!-- Buttons go here-->
     </header>
     <sheet>
         <!-- Content goes here: -->
         <field name="name"/>
         <field name="is_done"/>
     </sheet>
   </form>
```

### Adding action buttons 添加动作按钮
Forms can have buttons to run actions. These are able to trigger work ow actions, run Window Actions, such as opening another form, or run Python functions defined in the model.  

表单可以拥有运行动作的按钮。这些按钮能够触发工作流动作，运行Window动作，比如打开另外一个表单，或者是运行定义在模型中的函数：  

They can be placed anywhere inside a form, but for document-style forms, the recommended place for them is the `<header>` section.  

在表单内部这些按钮可以被方法任何地方，不过为了遵循文档风格的表单，我们建议还是将这些按钮放倒 `<header>` 区域。  

For our application, we will add two buttons to run methods of the `todo.task` model:  

对于我们的应用而言，我们需要添加两个按钮以运行`todo.task`模型的方法：  

```html
   <header>
     <button name="do_toggle_done" type="object"
       string="Toggle Done" class="oe_highlight" />
     <button name="do_clear_done" type="object"
       string="Clear All Done" />
   </header>
```

The basic attributes for a button are: string with the text to display on the button, the type of action it performs, and the name that is the identifier for that action. The optional class attribute can apply CSS styles, just like in regular HTML. 

按钮的基本属性是：含有文本的string会显示在按钮上————按钮要执行的动作类型，name用做动作的标识。可选的class属性合普通HTML一样可以应用CSS样式。  

### Organizing forms using groups 利用group来组织表单
The `<group>` tag allows organizing the form content. Placing `<group>` elements inside a `<group>` element creates a two column layout inside the outer group. Group elements are advised to have a name to make it easier for other modules to extend on them.  

`<group>`标签能够过组织表单内容。将`<group>`元素放倒一个`<group>` 元素中可以在外部的group中创建一个两列的布局。建议group元素包含一个能够过让其它模块能够在自身基础上进行相对容易的扩展。  


We will use this to better organize our content. Let's change the `<sheet>` content of our form to match this:  

我们使用group来更好地组织内容。我们来变更表单的`<sheet>`内容以匹配以下内容：  

```xml
     <sheet>
       <group name="group_top">
         <group name="group_left">
           <field name="name"/>
         </group>
         <group name="group_right">
           <field name="is_done"/>
           <field name="active" readonly="1"/>
         </group>
       </group>
     </sheet>
```

### The complete form view 完整的表单视图
At this point, our record in todo_view.xml for the todo.task form view should look like this:  

站在这个角度来说，用于todo.task表单视图的todo_view.xml的中的记录，应该是这个样子：  

```xml
<record id="view_form_todo_task" model="ir.ui.view">
      <field name="name">To-do Task Form</field>
      <field name="model">todo.task</field>
      <field name="arch" type="xml">
        <form>
          <header>
            <button name="do_toggle_done" type="object"
                    string="Toggle Done" class="oe_highlight" />
            <button name="do_clear_done" type="object"
                    string="Clear All Done" />
          </header>
          <sheet>
              <group name="group_top">
              <group name="group_left">
                <field name="name"/>
              </group>
              <group name="group_right">
                <field name="is_done"/>
                <field name="active" readonly="1" />
              </group>
            </group>
          </sheet>
        </form>

      </field>
   </record>
```

Remember that for the changes to be loaded into our Odoo database, a module upgrade is needed. To see the changes in the web client, the form needs to be reloaded: either click again on the menu option that opens it, or reload the browser page (F5 in most browsers).  

记住，如果要将变更载入到Odoo数据库中，模块也应该升级。为了在web客户端浏览变更，表单需要重新载入：在此点击打开的菜单选项，或者是重新加载页面（多数浏览器使用的是F5）。  

Now, let's add the business logic for the actions buttons.  

现在，让我们为动作按钮添加业务逻辑。  

## Adding list and search views 添加列表和搜索视图
When viewing a model in list mode, a `<tree>` view is used. Tree views are capable of displaying lines organized in hierarchies, but most of the time they are used to display plain lists.  

当在列表模式浏览一个模型时，使用的是`<tree>`视图。树形视图在分层结构中能够兼容又组织结构的显示行，不过很多时候它们被用来显示普通列表。  

We can add the following tree view definition to `todo_view.xml`:  

我们可以添加下面的的树形视图定义到`todo_view.xml`：  

```xml
<record id="view_tree_todo_task" model="ir.ui.view">
     <field name="name">To-do Task Tree</field>
     <field name="model">todo.task</field>
     <field name="arch" type="xml">
       <tree colors="gray:is_done==True">
         <field name="name"/>
         <field name="is_done"/>
       </tree>
     </field>
</record>
```

We have defined a list with only two columns, name and is_done. We also added a nice touch: the lines for done tasks (is_done==True) are shown in grey.  

我们定义了一个仅有两个列（name和is_done）的列表。我们也可以添加一个好看的按钮：完成任务的行（｀is_done==True｀）会显示为灰色。  

At the top right of the list Odoo displays a search box. The default fields it searches for and available predefined filters can be defined by a `<search>` view.  

在列表到右侧，Odoo显示了一个搜索框。默认的字段搜索的是事先定义好的，过滤器可以通过`<search>`视图来定义。  

As before, we will add this to the todo_view.xml:  

和之前一样，我们添加下面的内容到todo_view.xml：  

```xml
   <record id="view_filter_todo_task" model="ir.ui.view">
     <field name="name">To-do Task Filter</field>
     <field name="model">todo.task</field>
     <field name="arch" type="xml">
       <search>
         <field name="name"/>
         <filter string="Not Done"
                 domain="[('is_done','=',False)]"/>
         <filter string="Done"
                 domain="[('is_done','!=',False)]"/>
       </search>
     </field>
   </record>
```

The `<field>` elements define fields that are also searched when typing in the search box. The `<filter>` elements add predefined filter conditions, using domain syntax that can be selected with a user click.  

`<field>`元素定义了在搜索框中进行输入时可以被搜索的定义字段。`<filter>`元素添加之前定义过的过滤器条件，域语法的选择在用户点击时进行选择。  

## Adding business logic  添加业务逻辑
Now we will add some logic to our buttons. Edit the todo_model.py Python file to add to the class the methods called by the buttons. We will use the new API introduced in Odoo 8.0. For backward compatibility, by default Odoo expects the old API, and to create methods using the new API we need to use Python decorators on them. First we need to import the new API, so add it to the import statement at the top of the Python file:  

现在，我们要给按钮添加一些逻辑。编辑Python文件todo_model.py，然后添加可以被按钮调用的类方法。我们使用在Odoo 8.0中引入的新API。考虑到向后兼容性，默认Odoo使用的是旧版本的API，要创建使用新API的方法，我们需要对定义的方法使用Python装饰器。首先我们需要导入新API，

```python
from openerp import models, fields, api
```

The **Toggle Done**  button's action will be very simple: just toggle the Is Done? flag. For logic on a record, the simplest approach is to use the `@api.one` decorator. Here self will represent one record. If the action was called for a set of records, the API would handle that and trigger this method for each of the records.  

Toggle Done按钮的动作非常简单：切换是否完成？对于一条记录的逻辑来说，最简单的方法是使用装饰器`@api.one`。这里self表示的是一条记录。如果动作被一组记录调用，API就可以处理这样的调用，然后喂每一条记录出发这个方法。  

Inside the TodoTask class add:  

在TodoTask类内部添加以下内容：  

```python
@api.one
   def do_toggle_done(self):
       self.is_done = not self.is_done
       return True
```

As you can see, it simply modifies the is_done field, inverting its value. Methods, then, can be called from the client side and must always return something. If they return None, client calls using the XMLRPC protocol won't work. If we have nothing to return, the common practice is to just return the True value.  

如你所见，上面的方法简单的修改了is_done字段，并转换了该字段值。然后，方法就能够从客户端调用了，而且方法必须有返回的内容。如果，方法返回了None，那么使用XMLRPC协议的客户端调用就不会正常工作。如果我们没有需要返回的东西，常见的做法是返回真值。  

After this, if we restart the Odoo server to reload the Python file, the Toggle Done button should now work.  

这样做之后，如果我们重启Odoo服务器来读取Python文件，现在，Toggle Done这个按钮就可以使用了。  

For the **Clear All Done** button we want to go a little further. It should look for all active records that are done, and make them inactive. Form buttons are supposed to act only on the selected record, but to keep things simple we will do some cheating, and it will also act on records other than the current one:  

对于**Clear All Done**按钮我们想做的更近一步。它应该能够查询所有已完成的活动记录，而且能够将这些记录标记为不活动。表单按钮被假设为仅与被选择的记录交互，但是，简单起见，我们就偷懒一点儿，它也可以和其它的记录交互，而仅仅是当前的记录：  

```python
 @api.multi
   def do_clear_done(self):
       done_recs = self.search([('is_done', '=', True)])
       done_recs.write({'active': False})
       return True
```


On methods decorated with @api.multi the self represents a recordset. It can contain a single record, when used from a form, or several records, when used from a list view. We will ignore the self recordset and build our own done_recs recordset containing all the tasks that are marked as done. Then we set the active flag to False, in all of them.  

被@api.multi所装饰的方法的self表示一个记录集。在它被表单调用时，它可以包含一个单记录，或者是从列表视图调用时，包含多个记录。我们会忽略self的记录集，然后构建我们自己的包含了所有被标记为done的任务的记录集done_recs。接着，我们将记录集中的全部活动旗帜都设置为False。  

The search is an API method returning the records meeting some conditions. These conditions are written in a domain, that is a list of triplets. We'll explore domains in more detail later.  

search是一个返回匹配指定条件的记录集的API方法。这些条件被写在一个域中，即一个三元素的列表。稍后我们会学习域的更多细节。  

The write method sets values at once on all elements of the recordset. The values to write are described using a dictionary. Using write here is more efficient than iterating through the recordset to assign the value to them one by one.  

write方法一次性对记录的所有元素进行赋值。被写入的值使用字典来表示。这里使用write相比迭代整个记录再一个接一个的赋值效率更高。  

Note that `@api.one` is not the most efficient for these actions, since it will run for each selected record. The `@api.multi` ensures that our code runs only once even if there is more than one record selected when running the action. This could happen if an option for it were to be added on the list view.  

注意，对于这些动作来说`@api.one`并不是最有效率的，因为，它会执行每一条被选择记录。`@api.multi`可以保证我们的代码只运行一次，即便是再执行动作时存在多条被选择的记录。这样的事情在选项被添加到列表视图时可能就会发生。  

## Setting up access control security 设置访问控制安全
You might have noticed, upon loading our module is getting a warning message in the server log: **The model todo.task has no access rules, consider adding one**.  

你或许也注意到了，之前在载入模块时我们碰到了服务器日志中的错误警告： **The model todo.task has no access rules, consider adding one**.  

The message is pretty clear: our new model has no access rules, so it can't be used by anyone other than the admin super user. As a super user the admin ignores data access rules, that's why we were able to use the form without errors. But we must fix this before other users can use it.  

这条消息说得很清楚：我们的新模型还没有访问规则，因此模型可以被任何其它的非admin超级用户使用。作为超级用户，admin跳过了数据访问规则，这就是我们为什么能够过使用表单而不会碰到错误消息的原因。在其它用户使用这个漏洞之前，我们必须修复它。  

To get a picture of what information is needed to add access rules to a model, use the web client and go to: Settings|Technical|Security|Access Controls List.  

为了获取多模型添加访问规则所需的信息蓝图，我们可以使用web客户端依次访问：Settings|Technical|Security|Access Controls List。  

img:omit   

Here we can see the ACL for the mail.mail model. It indicates, per group, what actions are allowed on records.  

这里我们可以看到mail.mail模型的ACL。说明了，每一个组对记录都有什么动作可以执行。  

This information needs to be provided by the module, using a data file to load the
lines into the ir.model.access model. We will add full access on the model to the employee group. Employee is the basic access group nearly everyone belongs to.  

这些信息应由模块来提供，使用数据模型以载入行到ir.model.access模型。我们会给雇员组添加完整的访问权限。雇员是一个基本的访问组，几乎所有人都在其中。  

This is usually done using a CSV file named security/ir.model.access.csv. Models have automatically generated identi ers: for todo.task the identifier is model_todo_task. Groups also have identifiers set by the modules creating them. The employee group is created by the base module and has identifier base.group_user. The line's name is only informative and it's best if it's kept unique. Core modules usually use a dot-separated string with the model name and the group. Following this convention we would use todo.task.user.  

我们通常使用一个名称为security/ir.model.access.csv的CSV文件来完成操作。模型已经自动地生成了标识符：todo.task的标识符是model_todo_task。组通过创建自己的模块来设置标识符。employee组是由base模块创建，并拥有标识符base.group_user。行的名称是唯一提高信息的地方，所以你最好将名称唯一化。核型模块通常使用点号分隔的模型名和组。为了遵循这个约定我们接下来使用todo.task.user。  
Now we have everything we need to know, let's add the new file with the following content:  

现在，我们已经了解了所有需要了解的事情，接着将以下内容写入到新文件：  

```
   id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_
   unlink
   access_todo_task_group_user,todo.task.user,model_todo_task,base.group_
   user,1,1,1,1
```

We must not forget to add the reference to this new  le in the `__openerp__.py` descriptor's data attribute, so that should look like this:  

我们一定不能忘记在描述符`__openerp__.py`的数据属性中应用这个新文件，所以添加了引用的`__openerp__.py`文件是这个样子的：  

```python
   'data': [
       'todo_view.xml',
       'security/ir.model.access.csv',
   ],
```

As before, upgrade the module for these additions to take effect. The warning message should be gone, and you can con rm the permissions are OK by logging in with the user demo (password is also demo) and trying the to-do tasks feature.  

和之前一样，为了使这些变更生效你需要为这些附加的属性升级模块。这样操作之后，警告消息应该就看不到了，你可以同使用用户demo（密码也是demo）登录来检查权限，同时也可以体验to－do的应用功能。  

## Row-level access rules 多级别访问规则
Odoo is a multi-user system, and we would like the to-do tasks to be private to each user. Fortunately for us, Odoo also supports row-level access rules. In the Technical menu they can be found in the Record Rules option, alongside the Access Control List.  

Odoo是一个多用户系统，我们希望任务清单对每个用户都是私密的。非常幸运的是Odoo同时支持多级别的访问规则。在技术菜单中，它们可以和“访问控制列表”在一起的“纪录规则”选项中找到。  

Record rules are defined in the `ir.rule model`. As usual, we need a distinctive name. We also need the model they operate on and the domain to force access restriction. The domain filter uses the same domain syntax mentioned before, and used across Odoo.  

纪录规则被定义在了ir.rule模型中。通常，我们需要一个不同的名字。我们也需要模型可以操作，域强制执行访问限制。域了过滤器使用之前提到过的域语法，而且是在整个Odoo中都是如此使用。  

Finally, rules may be either global (the global field is set to True) or only for particular security groups. In our case, it could perfectly be a global rule, but to illustrate the most common case, we will make it a group-specific rule, applying only to the employees group.  

最后，规则可以是全局的（全局字段设置为True）或者只针对特殊的安全组。就我们的这个例子而言，可以完美地执行全局规则，不过为了说明最常见的情况，我们仅对雇员组应用组规则。  

We should create a security/todo_access_rules.xml file with this content:  

我们应该创建一个包含以下内容的security/todo_access_rules.xml文件：  

```xml
 <?xml version="1.0" encoding="utf-8"?>
   <openerp>
     <data noupdate="1">
       <record id="todo_task_user_rule" model="ir.rule">
           <field name="name">ToDo Tasks only for owner</field>
           <field name="model_id" ref="model_todo_task"/>
           <field name="domain_force">[('create_uid','=',user.id)]
           </field>
           <field name="groups" eval="[(4,ref('base.group_user'))]"/>
       </record>
   </data>
</openerp>
```

Notice the noupdate="1" attribute. It means this data will not be updated in module upgrades. This will allow it to be customized later, since module upgrades won't destroy user-made changes. But beware that this will also be so while developing, so you might want to set noupdate="0" during development, until you're happy with the data file.  

注意属性noupdate="1"。它的意思是该数据在模块升级时不会被更新。这个属性可以在后面自定义，因为模块的升级不会摧毁由用户产生的变更。而且在在开发时也确实会发生这样的事情，所以你可以在开发时设置noupdate="0"，直到你希望使用数据文件为止。  

In the groups field, you will also find a special expression. It's a one-to-many relational field, and they have special syntax to operate with. In this case, the (4, x) tuple indicates to append x to the records, and x is a reference to the employees group, identified by base.group_user.  

在group字段，你也会发现一个特殊的表达式。它是一个一对多关系字段，有特殊的语法来操作它们。本例中，(4, x)元组表示将x追加到记录，而且x是一个到通过base.group_user来识别的雇员组的引用。   

As before, we must add the file to `__openerp__.py` before it can be loaded to the module:  

就像之前那样，我们必须在`__openerp__.py`被载入到模块之前，对`__openerp__.py`添加规则文件：  

```python
'data': [
       'todo_view.xml',
       'security/ir.model.access.csv',
       'security/todo_access_rules.xml',
   ],
```

## Adding an icon to the module 对模块添加一个icon
Our module is looking good. Why not add an icon to it to make it look even better? For that we just need to add to the module a static/description/icon.png file with the icon to use.  

我们的模块看上去还好。那么，为什么不为模块添加一个图标，使它更好看？要实现这个目的，我们只需要为模块添加一个包含要用icon的static/description/icon.png文件。  

The following commands add an icon copied form the core *Notes* module:  

下面的命令添加了一个来自*Notes*模块的图标副本：  

```bash
$ mkdir -p ~/odoo-dev/custom-addons/todo_app/static/description
$ cd ~/odoo-dev/custom-addons/todo_app/static/description
$ cp ../odoo/addons/note/static/description/icon.png ./
```

Now, if we update the module list, our module should be displayed with the new icon.  

现在，若是我们更新模块列表，我们的模块应该可以显示这个新的图标。  

## Summary 总结
We created a new module from the start, covering the most frequently used elements in a module: models, the three base types of views (form, list, and search), business logic in model methods, and access security.  

我们从创建一个新模块开始，并且覆盖了在模块中最常使用的元素：模型，视图的的三个基本类型（表单、列表、和搜索），模型方法中的业务逻辑，以及访问安全。  

In the process, you got familiar with the module development process, which involves module upgrades and application server restarts to make the gradual changes effective in Odoo.  

在此期间，你已经熟悉了模块开发的过程，包括模块升级，以及为了在Odoo中让渐进式变更生效而重启应用服务器。  

Always remember, when adding model  elds, an upgrade is needed. When changing Python code, including the manifest file, a restart is needed. When changing XML or CSV files, an upgrade is needed; also when in doubt, do both: upgrade the modules and restart the server.  

永远不要忘记，在你添加模型字段时，必须升级模块。当改版Python代码，包括清单文件时，都需要对服务器重启。当变更XML或者CSV文件时，升级也是必须的。你要是有所怀疑的话，可以两个操作都执行：升级模块，然后重启服务器。

In the next chapter, you will learn about building modules that stack on existing ones to add features.  

在下一章，你会学习到如在已存在模块之上构建模块，以添加新功能。  