Chapter 7 ORM Application Logic – Supporting Business Processes
************

In this chapter, you will learn to write code to support business logic in your models and you will also learn how it can be activated on events and user actions. Using
the Odoo programming API, we can write complex logic and wizards allow us to provide a rich user interaction with these programs.  

## To-do wizard ToDo的引导程序
With the wizards, we can ask users to input information to be used in some processes. Suppose our to-do app users regularly need to set deadlines and the responsible persons for a large number of tasks. We could use an assistant to help them with this. It should allow them to pick the tasks to be updated and then choose the deadline date and/or the responsible user to set on them.  

使用引导程序，我们可以在部分业务过程中要求用户输入要用的的信息。假设我们的todo用户需要设置截止日期，以及负责大量任务的负责人。我们可以让一位助手来帮助它们完成需求。这个助手允许用户选择需要更新的任务，然后选择截止日期／或者设置负责人。  

We will start by creating a new module for this feature: todo_wizard. Our module will have a Python  le and an XML  le, so the `todo_wizard/__openerp__.py` description will be as shown in the following code:  

要实现这个功能，我们从创建一个新的模块开始：todo_wizard。我们的模块包含一个Python文件和一个XML文件，`todo_wizard/__openerp__.py`的内容描述一如下面代码所示：   

```python
 { 'name': 'To-do Tasks Management Assistant',
     'description': 'Mass edit your To-Do backlog.',
     'author': 'Daniel Reis',
     'depends': ['todo_user'],
     'data': ['todo_wizard_view.xml'], }
```


The `todo_wizard/__init__.py` file to load our code is just one line, as follows:  

`todo_wizard/__init__.py`文件将用一行语句载入代码：  

```python
from . import todo_wizard_model
```

Next, we need to describe the data model supporting our wizard.  

接下来，为了支持的引导程序，我们需要描述数据模型。  

### Wizard model 引导程序的模型
A wizard displays a form view to the user, usually in a dialog window, with some fields to be filled in. These will then be used by the wizard logic.  

引导程序向用户展示了一个表单视图，即，通常在一个对话窗口中填入部分字段。这些操作稍后会被引导程序逻辑所使用。  

This is implemented using the model/view architecture used for regular views, with a difference: the supporting model is based on models.TransientModel instead of models.Model.  

This type of model is also stored in the database, but the data is expected to be useful only until the wizard is completed or canceled. Server vacuum processes regularly clean up old wizard data from the corresponding database tables.
The todo_wizard/todo_wizard_model.py  le will de ne the three  elds we need: the lists of tasks to update, the user responsible for them, and the deadline to set on them, as shown here:  

```python
 # -*- coding: utf-8 -*-
   from openerp import models, fields, api
   from openerp import exceptions  # will be used in the code
   import logging
   _logger = logging.getLogger(__name__)

   
   class TodoWizard(models.TransientModel):
     _name = 'todo.wizard'
     task_ids = fields.Many2many('todo.task', string='Tasks')
     new_deadline = fields.Date('Deadline to Set')
     new_user_id = fields.Many2one(
       'res.users',string='Responsible to Set')
```


It's worth noting that if we used a one to many relation, we would have to add the inverse many to one  eld on to-do tasks. We should avoid many to one relations between transient and regular models, and so we used a many to many relation that ful lls the same purpose without the need to modify the to-do task model.  

We are also adding support to message logging. The logger is initialized with the two lines just before the TodoWizard, using the Python logging standard library. To write messages to the log we can use:  

```
   _logger.debug('A DEBUG message')
   _logger.info('An INFO message')
   _logger.warning('A WARNING message')
   _logger.error('An ERROR message')
```

We will see some usage examples in this chapter.  

### Wizard form 引导程序表单
The wizard form view looks exactly the same as regular forms, except for two specific elements:  

引导程序的表单视图看起来和普通视图完全一样，除了指定的两个元素之外：  

- A <footer> section can be used to place the action buttons.
- A special cancel button type is available to interrupt the wizard without
performing any action.  

- <footer>部分被用来放置动作按钮。  
- 特殊的取消按钮在不执行任意动作的情况下也可以中断引导程序。  

This is the content of our `todo_wizard/todo_wizard_view.xml` file:  

这里是`todo_wizard/todo_wizard_view.xml`文件的内容：  

```xml
<openerp>
     <data>
       <record id="To-do Task Wizard" model="ir.ui.view">
         <field name="name">To-do Task Wizard</field>
         <field name="model">todo.wizard</field>
         <field name="arch" type="xml">
           <form>
             <div class="oe_right">
               <button type="object" name="do_count_tasks"
                       string="Count" />
               <button type="object" name="do_populate_tasks"
                       string="Get All" />
             </div>
             <field name="task_ids" />
             <group>
               <group> <field name="new_user_id" /> </group>
               <group> <field name="new_deadline" /> </group>
             </group>
             <footer>
               <button type="object" name="do_mass_update"
                    string="Mass Update" class="oe_highlight"
                       attrs="{'invisible':
                               [('new_deadline','=',False),
                                ('new_user_id', '=',False)]}" />
               <button special="cancel" string="Cancel"/>
             </footer>
           </form>
         </field>
</record>
       <!-- More button Action -->
       <act_window id="todo_app.action_todo_wizard"
           name="To-Do Tasks Wizard"
           src_model="todo.task" res_model="todo.wizard"
           view_mode="form" target="new" multi="True" />
</data> 
</openerp>
```


The window action we see in the XML adds an option to the More button of the to-do task form by using the `src_model` attribute. target="new" makes it open as a dialog window.  



You might also have noticed attrs in the Mass Update button used to make it invisible until either a new deadline or responsible user is selected.  

你或许也注意到了，批量更新按钮中的attr是不用的，知道新的截止日期或者负责人其中的一个被中为止。  

This is how our wizard will look:  

这就是引导程序的外观：  

img:omit  

### Wizard business logic 引导程序的业务逻辑
Next we need to implement the actions performed while clicking on the Mass Update button. The method called by the button is do_mass_update and it should be de ned in the todo_wizard/todo_wizard_model.py  le, as shown in the following code:  

接下来，我们需要在点击批量更新按钮时实现动作的执行。

```python
 @api.multi
       def do_mass_update(self):
           self.ensure_one()
           if not (self.new_deadline or self.new_user_id):
             raise exceptions.ValidationError('No data to update!')
           # else:
           _logger.debug('Mass update on Todo Tasks %s',
               self.task_ids.ids)
           if self.new_deadline:
               self.task_ids.write({'date_deadline': self.new_deadline})
           if self.new_user_id:
               self.task_ids.write({'user_id': self.new_user_id.id})
return True
```

Our code can handle only one wizard instance at a time. We could have used @api.one, but it is not advised to do so in wizards. In some cases, we want the wizard to return a window action telling the client what to do next. That is not possible with @api.one, since it would return a list of actions instead of a single one.  

Because of this, we prefer to use @api.multi but then we use ensure_one() to check that self represents a single record. It should be noted that self is a record representing the data on the wizard form.  

The method begins by validating if a new deadline date or responsible user was given, and raises an error if not. Next, we demonstrate writing a message to the server log.  

If the validation passes, we write the new values given to the selected tasks. We are using the write method on a record set, such as the task_ids to many  eld to perform a mass update. This is more ef cient than repeating a write on each record in a loop.  

Now we will work on the logic behind the two buttons at the top: Count and Get All.  

### Raising exceptions
When something is not right, we will want to interrupt the program with an error message. This is done by raising an exception. Odoo provides a few additional exception classes to the ones available in Python. These are examples for the most useful ones:  

```python
   from openerp import exceptions
   raise exceptions.Warning('Warning message')
   raise exceptions.ValidationError('Not valid message')
```

The Warning message also interrupts execution but can sound less severe that a ValidationError. While it's not the best user interface, we take advantage of that on the Count button to display a message to the user:  

```python
       @api.multi
       def do_count_tasks(self):
           Task = self.env['todo.task']
           count = Task.search_count([])
           raise exceptions.Warning(
             'There are %d active tasks.' % count)
```

### Auto-reloading code changes
When you're working on Python code, the server needs to be restarted every time the code is changed to reload it. To make life easier for developers an --auto- reload option is available. It monitors the source code and automatically reloads it if changes are detected. Here is an example of it's usage:  

```shell
$ ./odoo.py -d v8dev --auto-reload
```

But this is a Linux-only feature. If you are using Debian/Ubuntu box to run the server, as recommended in Chapter 1, Getting Started with Odoo Development, it should work for you. The pyinotify Python package is required, and it should be installed either through apt-get or pip, as shown here:  

```shell
$ sudo apt-get install python-pyinotify  # using OS packages
$ pip install pyinotify  # using pip, possibly in a virtualenv
```

### Actions on the wizard dialog 引导程序对话上的动作
Now suppose we want a button to automatically pick all the to-do tasks to spare the user from picking them one by one. That's the point of having the Get All button in the form. The code behind this button will get a record set with all active tasks and assign it to the tasks in the many to many field.  

But there is a catch here. In dialog windows, when a button is pressed, the wizard window is automatically closed. We didn't face this problem on the Count button because it uses an exception to display it's message; so the action fails and the window is not closed.  

Fortunately we can work around this behavior by returning an action to the client that reopens the same wizard. The model methods are allowed to return an action for the web client to perform, in the form of a dictionary describing the window action to execute. This dictionary uses the same attributes used to de ne window actions in module XML.  

We will use a helper function for the window action dictionary to reopen the wizard window, so that it can be easily reused in several buttons, as shown here:  

```python
       @api.multi
       def do_reopen_form(self):
           self.ensure_one()
           return {
               'type': 'ir.actions.act_window',
               'res_model': self._name,  # this model
               'res_id': self.id,  # the current wizard record
               'view_type': 'form',
               'view_mode': 'form',
               'target': 'new'}
```

It is worth noting that the window action could be anything else, like jumping to a speci c form and record, or opening another wizard form to ask for additional user input.  

Now the Get All button can do its job and keep the user working on the same wizard:  

```python
  @api.multi
       def do_populate_tasks(self):
           self.ensure_one()
           Task = self.env['todo.task']
           all_tasks = Task.search([])
           self.task_ids = all_tasks
           # reopen wizard form on same wizard record
           return self.do_reopen_form()
```

Here we can see how to get a reference to a different model, which is todo.task in this case, to perform actions on it. The wizard form values are stored in the transient model and can be read and written as in regular models. We can also see that the method sets the task_ids value with the list of all active tasks.  

Note that since self is not guaranteed to be a single record, we validate that using self.ensure_one(). We shouldn't use the @api.one decorator because it would wrap the returned value in a list. Since the web client expects to receive a dictionary and not a list, it wouldn't work as intended.  

## Working with the server 使用Odoo服务器
Our server code will usually run inside a method of a model, as is the case for do_mass_update() in the preceding code.  

我们的服务端代码

In this context, self represents the recordset being acted upon. Instances of model classes are actually recordsets. For actions executed from views, this will be only the record currently selected on it. If it's a form view, it is usually a single record, but in tree views, there can be several records.  

在这个上下文中，self表示在启动时被使用记录集。模型类的实例实际上是记录集。为了从视图执行动作，它

The self.env object allows us to access our running environment; this includes the information on the current session, such as the current user and session context, and also access all the other models available in the server.  

self.env对象允许我们访问正在运行的环境；它包括当前回会话信息，比如当前用户和会话上下文，而且还可以访问服务端中所有可用的模型。  

To better explore programming on the server side, we can use the server interactive console, where we have an environment similar to what we can find inside a model method.  

为了更好地浏览服务端的程序，我们可以使用服务端的交互式控制台，在控制台中我们拥有了一个和我们字模型方法内可以找的东西相类似的环境。  

This is a new feature for version 9. It has been back-ported as a module for version 8, and it can be downloaded from the link `https://www.odoo.com/apps/modules/8.0/ shell/`. It just needs to be placed somewhere in your add-ons path, and no further installation is necessary, or you can use the following commands to get the code from GitHub and make the module available in our custom add-ons directory:  

这是版本9中的新功能。它已经移植到了版本8中，你也可以从链接`https://www.odoo.com/apps/modules/8.0/ shell/`下载。它需放到addons路径中，而且也不必安装，或者你可以使用下面的命令从Github上获取代码，然后让位于自定义addons目录中的模块启用：  

```shell
$ cd ~/odoo-dev
$ git clone https://github.com/OCA/server-tools.git -b 8.0
$ ln -s server-tools/shell custom-addons/shell
$ cd ~/odoo-dev/odoo
```

To use this, run odoo.py with the shell command and the database to use as shown here:  

要完成以上操作，使用shell命令运行odoo.py，数据库的使用如下：  

```shelll
$ ./odoo.py shell -d v8dev
```

You will see the server start up sequence in the terminal ending in a >>> Python prompt. Here, self represents the record for the administrator user as shown here:  

你会在终端中以服务器启动序列，>>> 结尾的Python提示符

```shell
>>> self
res.users(1,)
>>> self.name
u'Administrator'
>>> self._name
'res.users'
>>> self.env
<openerp.api.Environment object at 0xb3f4f52c>
```

In the session above, we do some inspection on our environment. self represents a res.users recordset containing only the record with ID 1 and name Administrator. We can also confirm the recordset's model name inspecting self._name, and confirm that self.env is a reference for the environment.   

上面的回话中，我们对环境进行了检查。self表示一个仅包含ID为1，名字为Administrator的记录的记录集res.users。我们也可以透过检查self._name来验证记录集的模型名称，而且也可以证实self.env是一个对环境的引用。  

As usual, you can exit the prompt using Ctrl + D. This will also close the server process and bring you back to the system shell prompt.
The Model class referenced by self is in fact a recordset, an iterable collection of records. Iterating through a recordset returns individual records.  

通常，你可以使用Ctrl + D退出终端。这样做也将关闭服务进程，并返回到系统的shell提示符环境中。在真实的记录集，以及一个可迭代的记录集合中，模型类通可以过self来引用。 对一个记录集进行迭代将返回单个记录。  

The special case of a recordset with only one record is called a singleton. Singletons behave like records, and for all practical purposes are the same thing as a record. This particularity means that a record can be used wherever a recordset is expected.
Unlike multi-element recordsets, singletons can access their  elds using the dot notation, as shown here:  

仅有一个记录的的特殊记录集被称作`单记录`。单记录的行为何记录相似，在实际操作上也和记录一样。其特别的意义在于能使用记录集的地方就能使用记录。和多元素记录集不同，单记录可以使用点标记来访问自身对应的字段，一如下面内容所示：  

```
>>> print self.name
Administrator
>>> for rec in self:
    print rec.name
Adminstrator
```

### Using relation fields 使用关联字段
As we saw earlier, models can have relational fields: many to one, one to many, and many to many. These field types have recordsets as values.  

就像我们之前所看到的那样，模型拥有关联字段：多对一，一对多，以及多对多。这些字段都拥有记录集构成的值。  

In the case of many to one, the value can be a singleton or an empty recordset. In both cases, we can directly access their field values. As an example, the following instructions are correct and safe:  

在多对一的情况中，值可以是单个的，或者是一个空的记录集。这两种情况中，我们都可以直接地访问模型字段的值。例如，下面的命令都是正确且安全的：  

```python
>>> self.company_id
res.company(1,)
>>> self.company_id.name
u'YourCompany'
>>> self.company_id.currency_id
res.currency(1,)
>>> self.company_id.currency_id.name
u'EUR'
```

Conveniently, an empty recordset also behaves like a singleton, and accessing its fields does not return an error but just returns False. Because of this, we can traverse records using dot notation without worrying about errors from empty values, as shown here:  

很方便的是，空记录集的行为也和单个值相似，访问这些字段并不会返回错误，而是返回False。因此，我们使用点标记访问记录，而不用担心会有错误从空值返回，一如下面所示：  

```python
>>> self.company_id.country_id
res.country()
>>> self.company_id.country_id.name
False
```

### Querying models 查询模型
With self we can only access the method's recordset. But the self.env environment reference allows us to access any other model.  

使用self我们可以只访问方法到记录集。不过使用self.env的环境应用可以让我们访问任意其他的模型。  

For example, `self.env['res.partner']` returns a reference to the Partners model (which is actually an empty recordset). We can then use search() or browse() on it to generate recordsets.  

例如，放回了一个Partner模型的引用。我们可以在返回的引用之上去生成记录集。  

The `search()` method takes a domain expression and returns a recordset with
the records matching those conditions. An empty domain `[]` will return all records. If the model has the active special  eld, by default only the records with active=True will be considered. A few optional keyword arguments are available, as shown here:  

`search()` 方法接受一个域表达式并返回一个拥有匹配查询条件的记录构成的记录集。一个空的域`[]`将返回所有的记录。如果模型拥有特殊字段active，默认只考虑active=True的记录。如下，是三个可供选择的关键字参数：  

- order: This is a string to be used as the ORDER BY clause in the database query. This is usually a comma-separated list of field names. 
- limit: This sets a maximum number of records to retrieve.  
- offset: This ignores the first n results; it can be used with limit to query blocks of records at a time.  

- order：这是一个在数据库中进行查询时被当作ORDER BY子句来使用。  
- limit：该参数设置一个重新取回记录的最大数。  
- offset：使用该参数将忽略第一个结果；它可以用来在运行时限制对记录块的查询。  

Sometimes we just need to know the number of records meeting certain conditions. For that we can use search_count(), which returns the record count instead of
a recordset.  

有时候我仅需知道符合某些条件的记录数量。因此，我们可以使用search_count()，它返回的是记录数量而不是记录集的数量。  

The browse() method takes a list of IDs or a single ID and returns a recordset with those records. This can be convenient for the cases where we already know the IDs of the records we want.  

browse()方法接受一个ID列表或单个ID，然后返回包含这些记录的记录集。这在我们已经知道想要的记录ID时会非常方便。  

Some usage examples of this are shown here:  

该方法的使用示例如下：  

```shell
>>> self.env['res.partner'].search([('name', 'like', 'Ag')])
res.partner(7, 51)
>>> self.env['res.partner'].browse([7, 51])
res.partner(7, 51)
```

### Writing on records 写记录
Recordsets implement the active record pattern. This means that we can assign values on them, and these changes will be made persistent in the database. This is an intuitive and convenient way to manipulate data, as shown here:  

记录集实现了活动记录模式。这就意味着我们可以对它们赋值，然后这些变更将固化到数据中。

```shell
>>> admin = self.env['res.users'].browse(1)
>>> admin.name = 'Superuser'
>>> print admin.name
Superuser
```

Recordsets have three methods to act on their data: create(), write(), and unlink().  

记录集拥有三个作用数据的方法：create(), write(), 和 unlink()。  

The create() method takes a dictionary to map fields to values and returns the created record. Default values are automatically applied as expected, which is shown here:  

create()方法接受一个映射字段到值的字典，然后返回创建的记录。如下所示，默认值会如你所期望的那样自动地应用：  

```shell
>>> Partner = self.env['res.partner']
>>> new = Partner.create({'name': 'ACME', 'is_company': True})
>>> print new
res.partner(72,)
```

The unlink() method deletes the records in the recordset, as shown here:  

如下所示，unlink()方法删除记录集中的记录：  

```shell
>>> rec = Partner.search([('name', '=', 'ACME')])
>>> rec.unlink()
True
```

The write() method takes a dictionary to map fields to values. These are updated on all elements of the recordset and nothing is returned, as shown here:  

write()方法接受一个映射字段到值的字典。如下所示，

```shell
>>> Partner.write({'comment': 'Hello!'})
```

Using the active record pattern has some limitations; it updates only one field at a time. On the other hand, the write() method can update several fields of several records at the same time by using a single database instruction. These differences should be kept in mind for the cases where performance can be an issue.  

使用活动记录模式存在一点儿限制；该模式一次仅更新一个记录。换句话来说，write()方法可以使用数据库指令同时更新多个记录的多个字段。你应该将这些区别牢记在心，特别是遇到性能问题的时候。  

It is also worth mentioning copy() to duplicate an existing record; it takes that as an optional argument and a dictionary with the values to write on the new record. For example, to create a new user copying from the Demo User:  

使用copy()复制一个存在的记录也值得一提；该方法接受一个可选参数，以及一个拥有可以写新记录的值的字典。例如，创建一个拷贝自Demo用户的新用户：  

```shell
>>> demo = self.env.ref('base.user_demo')
>>> new = demo.copy({'name': 'Daniel', 'login': 'dr', 'email':''})
>>> self.env.cr.commit()
```

Remember that fields with the copy=False attribute won't be copied.  

记住拥有copy=False属性的字段不能被复制的。  

## Transactions and low-level SQL事务与底层SQL
Database writing operations are executed in the context of a database transaction. Usually we don't have to worry about this as the server takes care of that while running model methods.  

数据库的写操作是在数据库事务的上下文中执行的。通常在运行模型方法时，我们们不需要担心这类事件的。  

But in some cases, we may need a finer control over the transaction. This can be done through the database cursor self.env.cr, as shown here:  

不过，在某些情况下，我们想要更好的控制事务。如下所示，这个操作可以通过应用数据游标self.env.cr来实现：  

- self.env.cr.commit(): This commits the transaction's buffered write operations.
- self.env.savepoint(): This sets a transaction savepoint to rollback to.
- self.env.rollback(): This cancels the transaction's write operations since the last savepoint or all if no savepoint was created.

- self.env.cr.commit()：该方法将提交事务临时写操作。  
- self.env.savepoint()：该方法设置一个用来回滚的保存点。  
- self.env.rollback()：该方法将取消上次保存点或者所有的未创建保存点的操作的写操作。  

>### Tips 提示
>In a shell session, your data manipulation won't be made effective in the database until you use self. env.cr.commit().  
>在Shell会话中，你对数据操作在应用self.env.cr.commit()之前，是不会向数据写入数据的。  

With the cursor execute() method, we can run SQL directly in the database. It takes a string with the SQL statement to run and a second optional argument with a tuple or list of values to use as parameters for the SQL. These values will be used where %s placeholders are found.  

应用游标的execute()，我们可以在数据库中直接运行SQL。

If you're using a SELECT query, records should then be fetched. The fetchall() function retrieves all the rows as a list of tuples and dictfetchall() retrieves them as a list of dictionaries, as shown in the following example:  

如果你在使用SELECT查询，记录就应该被取回。fetchall()函数重新取回由tuples列表形式的所有行，而dictfetchall()则取回这些行的一个字典组成的列表，一如下面的例子所示：   

```python
>>> self.env.cr.execute("SELECT id, login FROM res_users WHERE login=%s OR id=%s", ('demo', 1))
>>> self.env.cr.fetchall()
      [(4, u'demo'), (1, u'admin')]
```

It's also possible to run data manipulation language instructions (DML)
such as UPDATE and INSERT. Since the server keeps data caches, they may become inconsistent with the actual data in the database. Because of that, while using raw DML, the caches should be cleared afterwards by using self.env. invalidate_all().  

这里也可以运行数据操作语言指令（DML），比如UPDATE和INSERT。因此，服务器将数据缓存了起来，所以当前操作的数据会和位于数据库中的实际数据事不一致的。因此，当你使用原始的DML指令时，  

>### Caution!
>Executing SQL directly in the database can lead to inconsistent data. You should use it only if you are sure of what you are doing.  

>### 警告！
>在数据库中直接地执行SQL会导致数据不一致。你应该仅在自己要做的事情有个确切了解之后再进行操作。  

### Working with time and dates 使用时间和日期
For historical reasons, date and datetime values are handled as strings instead of the corresponding Python types. Also datetimes are stored in the database in UTC time. The formats used in the string representation are de ned by:  

由于历史原因，date和datetime被当做字符而不是相对应的Python对象。而且dateime在数据库使用UTC时间来存储。

- openerp.tools.misc.DEFAULT_SERVER_DATE_FORMAT
- openerp.tools.misc.DEFAULT_SERVER_DATETIME_FORMAT  

They map to `%Y-%m-%d` and `%Y-%m-%d %H:%M:%S` respectively.  

它们分别地映射为`%Y-%m-%d` 和 `%Y-%m-%d %H:%M:%S`。  

To help handle dates, `fields.Date` and `fields.Datetime` provide a few functions.  

为了处理日期，`fields.Date` 和 `fields.Datetime`提供了一个新函数。  

For example:  

```python
>>> from openerp import fields
>>> fields.Datetime.now()
'2014-12-08 23:36:09'
>>> fields.Datetime.from_string('2014-12-08 23:36:09')
    datetime.datetime(2014, 12, 8, 23, 36, 9)
```

Given that dates and times are handled and stored by the server in a naive UTC format, which is not time zone aware and is probably different from the time zone that the user is working on, a few other functions that help to deal with this are shown here:  

给定的日期和时间由服务器以天然的UTC格式来处理和存储的，由于对时区不敏感，可能服务器上的时区不同于用户所使用的时区，如下所示，这里有几个帮助我们处理这个问题的函数：  

- `fields.Date.today()`: This returns a string with the current date in the format expected by the server and using UTC as a reference. This is adequate to compute default values.  

- `fields.Datetime.now()`: This returns a string with the current datetime in the format expected by the server using UTC as a reference. This is adequate to compute default values.  

- `fields.Date.context_today(record, timestamp=None)`: This returns a string with the current date in the session's context. The timezone value is taken from the record's context, and the optional parameter to use is datetime instead of the current time.  

- `fields.Datetime.context_timestamp(record, timestamp)`:
That converts a naive datetime (without timezone) into a timezone aware datetime. The timezone is extracted from the record's context, hence the name of the function.  

- `fields.Date.today()`： 该方法返回一个

To facilitate conversion between formats, both fields.Date and fields.Datetime objects provide these functions:  

为了便于在各种格式之间进行转换，fields.Date和fields.Datetime对象都提供了这些功能：  

- `from_string(value)`: This converts a string into a date or datetime object.  
- `to_string(value)`: This converts a date or datetime object into a string in
the format expected by the server.  

- `from_string(value)`：该方法将字符串转换到date或者datetime对象。
- `to_string(value)`: 该方法将date、datetime转换到服务器所希望使用字符串格式。

### Working with relation fields 使用关联字段
While using the active record pattern, relational fields can be assigned recordsets.  

当使用活动记录模式，关联字段可以被赋值为记录集。  

For a many to one field, the value assigned must be a single record (a singleton recordset).  

对于多对一字段，值必须赋值为一个单记录（单元素的记录集）。  

For to-many fields, their value can also be assigned with a recordset, replacing the list of linked records, if any, with a new one. Here a recordset with any size is allowed.  

对于to－many字段，它们的值也可以被赋值为一个记录集，如果有可能的话，使用一个新的替换

While using the `create()` or `write()` methods, where values are assigned using dictionaries, relational fields can't be assigned to recordset values. The corresponding ID, or list of IDs should be used.  

当使用 `create()` 或者 `write()`方法时，值的赋值使用字典，关联字段不能够被赋值为记录集值。应该使用对应的ID，或者ID列表。  

For example, instead of `self.write({'user_id': self.env.user})`, we should rather use `self.write({'user_id': self.env.user.id})`.  

例如，我们应当使用`self.write({'user_id': self.env.user.id})`，而不是使用`self.write({'user_id': self.env.user})`。  

### Manipulating recordsets 操作记录集
We will surely want to add, remove, or replace the elements in these related fields, and so this leads to the question: how can recordsets be manipulated?  

想必，你想要在这些关联字段中添加或者替换元素，那么问题来了：如何

Recordsets are immutable but can be used to compose new recordsets. Some set operations are supported, which are shown here:  

记录集是可变的，不过也可以被用来生成新的记录集。如下所示，下面是一些被支持的操作：  

- `rs1 | rs2`: This results in a recordset with all elements from both recordsets.
- `rs1 + rs2`: This also concatenates both recordsets into one.
- `rs1 & rs2`: This results in a recordset with only the elements present in both recordsets.
- `rs1 - rs2`: This results in a recordset with the rs1 elements not present in rs2.


The slice notation can also be used, as shown here:  

如下所示，切片标记也是可以使用的：  

- `rs[0]` and `rs[-1]` retrieve the first element and the last elements.  

- `rs[1:]` results in a copy of the recordset without the first element. This yields the same records as `rs – rs[0]` but preserves their order.  

In general, while manipulating recordsets, you should assume that the record order is not preserved. However, addition and slicing are known to keep record order.  

通常，当操作记录集时，你应该

We can use these recordset operations to change the list by removing or adding elements. You can see this in the following example:  

- `self.task_ids |= task1`: This adds task1 element if it's not in the recordset.  

- `self.task_ids -= task1`: This removes the reference to task1 if it's present
in the recordset.  

- `self.task_ids = self.task_ids[:-1]`: This unlinks the last record.  

While using the `create()` and `write()` methods with values in a dictionary, a special syntax is used to modify to many  elds. This was explained in Chapter 4, Data Serialization and Module Data, in the section Setting values for relation  elds. Refer to the following sample operations equivalent to the preceding ones using `write()`:  

- `self.write([(4, task1.id, False)])`: This adds task1 to the member.  

- `self.write([(3, task1.id, False)])`: This unlinks task1.  

- `self.write([(3, self.task_ids[-1].id, False)])`: This unlinks the last element.  

### Other recordset operations 其他记录集操作
Recordsets support additional operations on them.  

记录集也支持额外的操作。  

We can check if a record is included or is not in a recordset by doing the following:  

我们可以通过下面的命令来检查记录是否包含在记录集中：  

- record in recordset  
- record not in recordset  

These operations are also available:  
 
这些操作也是可以使用的：  

- recordset.ids: This returns the list with the IDs of the recordset elements.  
- recordset.ensure_one(): This checks if it is a single record (singleton);
if it's not, it raises a ValueError exception.  
- recordset.exists(): This returns a copy with only the records that still exist.
- recordset.filtered(func): This returns a filtered recordset.
- recordset.mapped(func): This returns a list of mapped values.
- recordset.sorted(func): This returns an ordered recordset.

- recordset.ids: 返回记录集中元素的由ID组成列表。  
- recordset.ensure_one()：

Here are some usage examples for these functions:  

```shell
>>> rs0 = self.env['res.partner'].search([])
>>> len(rs0)  # how many records?
68
>>> rs1 = rs0.filtered(lambda r: r.name.startswith('A'))
>>> print rs1
res.partner(3, 7, 6, 18, 51, 58, 39)
>>> rs2 = rs1.filtered('is_company')
>>> print rs2
res.partner(7, 6, 18)
>>> rs2.mapped('name')
[u'Agrolait', u'ASUSTeK', u'Axelor']
>>> rs2.mapped(lambda r: (r.id, r.name))
[(7, u'Agrolait'), (6, u'ASUSTeK'), (18, u'Axelor')]
>> rs2.sorted(key=lambda r: r.id, reverse=True)
res.partner(18, 7, 6)
```

### The execution environment
The environment provides contextual information used by the server. Every recordset carries its execution environment in self.env with these attributes:  

- env.cr: This is the database cursor being used.  
- env.uid: This is the ID for the session user.  
- env.user: This is the record for the session user.  
- env.context: This is an immutable dictionary with a session context.  

The environment is immutable, and so it can't be modi ed. But we can create modi ed environments and then run actions using them. These methods can be used for that:  

- `env.sudo(user)`: If this is provided with a user record, it returns an environment with that user. If no user is provided, the administrator superuser will be used, which allows running specific queries bypassing security rules.  

- `env.with_context(dictionary)`: This replaces the context with a new one.  

- `env.with_context(key=value,...)`: This sets values for keys in the current
context.  

The `env.ref()` function takes a string with an External ID and returns a record for it, as shown here:  

`env.ref()`函数接受一个拥有扩展ID的字符串，返回一个

```shell
>>> self.env.ref('base.user_root')
res.users(1,)
```

## Model methods for client interaction 用户客户端交互的模型方法
We have seen the most important model methods used to generate recordsets and how to write on them. But there are a few more model methods available for more speci c actions, as shown here:  

- `read([fields])`: This is similar to browse, but instead of a recordset, it returns a list of rows of data with the fields given as it's argument. Each row is a dictionary. It provides a serialized representation of the data that can be sent through RPC protocols and is intended to be used by client programs and not in server logic.  

- `search_read([domain], [fields], offset=0, limit=None, order=None)`: This performs a search operation followed by a read on the resulting record list. It is intended to be used by RPC clients and saves them the extra round trip needed when doing a search first and then a read.  

- `load([fields], [data])`: This is used to import data acquired from a CSV file. The first argument is the list of fields to import, and it maps directly to a CSV top row. The second argument is a list of records, where each record is a list of string values to parse and import, and it maps directly to the CSV data rows and columns. It implements the features of CSV data import described in *Chapter 4, Data Serialization and Module Data*, like the External IDs support. It is used by the web client Import feature. It replaces the deprecated `import_data` method.  

- `export_data([fields], raw_data=False)`: This is used by the web client Export function. It returns a dictionary with a data key containing the data–a list of rows. The field names can use the .id and /id suffixes used in CSV files, and the data is in a format compatible with an importable CSV file. The optional raw_data argument allows for data values to be exported with their Python types, instead of the string representation used in CSV.  

The following methods are mostly used by the web client to render the user interface and perform basic interaction:  

- `name_get()`: This returns a list of (ID, name) tuples with the text representing each record. It is used by default to compute the `display_name` value, providing the text representation of relation fields. It can be extended to implement custom display representations, such as displaying the record code and name instead of only the name.  

- `name_search(name='', args=None, operator='ilike', limit=100)`: This also returns a list of (ID, name) tuples, where the display name matches the text in the name argument. It is used by the UI while typing in a relation field to produce the list suggested records matching the typed text. It is used to implement product lookup both by name and by reference while typing in a field to pick a product.  

- `name_create(name)`: This creates a new record with only the title name to use for it. It is used by the UI for the quick-create feature, where you can quickly create a related record by just providing its name. It can be extended to provide specific defaults while creating new records through this feature.  

- `default_get([fields])`: This returns a dictionary with the default values for a new record to be created. The default values may depend on variables such as the current user or the session context.  

- `fields_get()`: This is used to describe the model's field definitions, as seen in the View Fields option of the developer menu.  

- `fields_view_get()`: This is used by the web client to retrieve the structure of the UI view to render. It can be given the ID of the view as an argument or the type of view we want using `view_type='form'.` Look at an example of this: `rset.fields_view_get(view_type='tree')`.  

### Overriding the default methods
We have learned about the standard methods provided by the API. But what we can do with them doesn't end there! We can also extend them to add custom behavior to our models.  

The most common case is to extend the create() and write() methods. This can be used to add the logic triggered whenever these actions are executed. By placing our logic in the appropriate section of the custom method, we can have the code run before or after the main operations are executed.  

Using the TodoTask model as an example, we can make a custom create(), which would look like this:  

```python
  @api.model
   def create(self, vals):
       # Code before create
       # Can use the `vals` dict
       new_record = super(TodoTask, self).create(vals)
       # Code after create
       # Can use the `new` record created
       return new_record
```

A custom write() would follow this structure:  

```python
   @api.multi
   def write(self, vals):
       # Code before write
       # Can use `self`, with the old values
       super(TodoTask, self).write(vals)
       # Code after write
       # Can use `self`, with the new (updated) values
       return True
```

These are common extension examples, but of course any standard method available for a model can be inherited in a similar way to add to it our custom logic.  

These techniques open up a lot of possibilities, but remember that other tools are also available that are better suited for common speci c tasks and should be preferred:  

- To have a field value calculated based on another, we should use computed fields. An example of this is to calculate a total when the values of the lines are changed.  

- To have field default values calculated dynamically, we can use a field default bound to a function instead of a scalar value.  

- To have values set on other fields when a field is changed, we can use on-change functions. An example of this is when picking a customer to
set the document's currency to the corresponding partner's, which can afterwards be manually changed by the user. Keep in mind that on-change only works on form view interaction and not on direct write calls.  

- For validations, we should use constraint functions decorated with `@api. constrains(fld1,fld2,...)`. These are like computed fields but are expected to raise errors when conditions are not met instead of computing values.  

### Model method decorators
During our journey, the several methods we encountered used API decorators like @api.one. These are important for the server to know how to handle the method. We have already given some explanation of the decorators used; now let's recap the ones available and when they should be used:  

- @api.one: This feeds one record at a time to the function. The decorator does the recordset iteration for us and self is guaranteed to be a singleton. It's the one to use if our logic only needs to work with each record. It also aggregates the return values of the function on each record in a list, which can have unintended side effects.  

- @api.multi: This handles a recordset. We should use it when our logic can depend on the whole recordset and seeing isolated records is not enough, or when we need a return value that is not a list like a dictionary with a window action. In practice it is the one to use most of the time as @api.one has some overhead and list wrapping effects on result values.  

- @api.model: This is a class-level static method, and it does not use
any recordset data. For consistency, self is still a recordset, but its content is irrelevant.  

- @api.returns(model): This indicates that the method return instances of the model in the argument, such as res.partner or self for the current model.  

The decorators that have more speci c purposes that were explained in detail in Chapter 5, Models – Structuring Application Data are shown here:  

- `@api.depends(fld1,...)`: This is used for computed field functions to identify on what changes the (re)calculation should be triggered.  

- `@api.constrains(fld1,...)`: This is used for validation functions to identify on what changes the validation check should be triggered. 

- `@api.onchange(fld1,...)`: This is used for on-change functions to identify the fields on the form that will trigger the action.  

In particular the on-change methods can send a warning message to the user interface. For example, this could warn the user that the product quantity just entered is not available on stock, without preventing the user from continuing. This is done by having the method return a dictionary describing the following warning message:  

```python
           return {
               'warning': {
                   'title': 'Warning!',
                   'message': 'The warning text'}
}
```

### Debugging
We all know that a good part of a developer's work is to debug code. To do this we often make use of a code editor that can set breakpoints and run our program step by step. Doing so with Odoo is possible, but it has it's challenges.  

If you're using Microsoft Windows as your development workstation, setting up an environment capable of running Odoo code from source is a nontrivial task. Also the fact that Odoo is a server that waits for client calls and only then acts on them makes it quite different to debug compared to client-side programs.  

While this can certainly be done with Odoo, arguably it might not be the most pragmatic approach to the issue. We will introduce some basic debugging strategies, which can be as effective as many sophisticated IDEs with some practice.  

Python's integrated debug tool pdb can do a decent job at debugging. We can set a breakpoint by inserting the following line in the desired place:  

```python
import pdb; pdb.set_trace()
```

Now restart the server so that the modi ed code is loaded. As soon as the program execution reaches that line, a (pdb) Python prompt will be shown in the terminal window where the server is running, waiting for our input.  

This prompt works as a Python shell, where you can run any expression or command in the current execution context. This means that the current variables can be inspected and even modi ed. These are the most important shortcut commands available:  

- h: This is used to display a help summary of the pdb commands.
- p: This is used to to evaluate and print an expression.
- pp: This is for pretty print, which is useful for larger dictionaries or lists.
- l: This lists the code around the instruction to be executed next.
- n (next): This steps over to the next instruction.
- s (step): This steps into the current instruction.
- c (continue): This continues execution normally.
- u(up): This allows to move up the execution stack.
- d(down): This allows to move down the execution stack.

The Odoo server also supports the `--debug` option. If it's used, when the server finds an exception, it enters into a *post mortem* mode at the line where the error was raised. This is a `pdb` console and it allows us to inspect the program state at the moment where the error was found.  

It's worth noting that there are alternatives to the Python built-in debugger. There is `pudb` that supports the same commands as `pdb` and works in text-only terminals, but uses a more friendly graphical display, making useful information readily available such as the variables in the current context and their values.  

img:omit  

It can be installed either through the system package manager or through pip, as shown here:  

```shell
$ sudo apt-get install python-pudb  # using OS packages
$ pip install pudb  # using pip, possibly in a virtualenv
```

It works just like pdb; you just need to use pudb instead of pdb in the breakpoint code.  

Another option is the Iron Python debugger ipdb, which can be installed by using the following code:  

```shell
$ pip install ipdb
```

Sometimes we just need to inspect the values of some variables or check if some
code blocks are being executed. A Python print statement can perfectly do the job without stopping the execution  ow. As we are running the server in a terminal window, the printed text will be shown in the standard output. But it won't be stored to the server log if it's being written to a file.  

Another option to keep in mind is to set debug level log messages at sensitive points of our code if we feel that we might need them to investigate issues in a deployed instance. It would only be needed to elevate that server logging level to DEBUG and then inspect the log fies.  

## Summary
In the previous chapters, you saw how to build models and design views. Here you went a little further learning how to implement business logic and use recordsets to manipulate model data.  

You also saw how the business logic can interact with the user interface and learned to create wizards that dialogue with the user and serve as a platform to launch advanced processes.  

In the next chapter, our focus will go back to the user interface, and you will learn how to create powerful kanban views and design your own business reports.  
