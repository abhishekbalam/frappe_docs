---
title: Hooks
image: /assets/frappe_io/images/frappe-framework-logo-with-padding.png
metatags:
 description: >
  Hooks allow you to "hook" into functionality and events of core parts of the Frappe Framework.
---

# Hooks

Hooks allow you to "hook" into functionality and events of core parts of the
Frappe Framework. This page documents all of the hooks provided by the framework.

## App Meta Data

These are automatically generated when you create a new app. Most of the time you
don't need to change anything here.

1. `app_name` - slugified name of the app
2. `app_title` - presentable app name
3. `app_publisher`
4. `app_description`
5. `app_version`
6. `app_icon`
7. `app_color`

## Javascript / CSS Assets

The following hooks allow you to inject static JS and CSS assets in various
 parts of your site.

### Desk

These hooks allow you to inject JS / CSS in `desk.html` which renders the
[Desk](/docs/user/en/desk).

```py
# injected in desk.html
app_include_js = "assets/js/app.min.js"
app_include_css = "assets/js/app.min.css"

# All of the above support a list of paths too
app_include_js = ["assets/js/app1.min.js", "assets/js/app2.min.js"]
```

### Portal

These hooks allow you to inject JS / CSS in `web.html` which renders the
[Portal](/docs/user/en/portal-pages).

```py
# injected in the web.html
web_include_js = "assets/js/app-web.min.js"
web_include_css = "assets/js/app-web.min.css"
# All of the above support a list of paths too
web_include_js = ["assets/js/web1.min.js", "assets/js/web2.min.js"]
```

### Web Form

These hooks allow you to add inject static JS and CSS assets in `web_form.html`
which is used to render Web Forms. These will work only for Standard Web Forms.

```py
webform_include_js = {"ToDo": "public/js/custom_todo.js"}
webform_include_css = {"ToDo": "public/css/custom_todo.css"}
```

> For user created Web Forms, you can directly write the script in the form
> itself.

### Page

These hooks allow you to inject JS assets in Standard Desk Pages.

```py
page_js = {"page_name" : "public/js/file.js"}
```

## Install Events

These hooks allow you to run code before and after installation of your app. For
example, [ERPNext](https://github.com/frappe/erpnext) has these
[defined](https://github.com/frappe/erpnext/blob/develop/erpnext/hooks.py#L37-L38).

```py
# python module path
before_install = "app.setup.install.before_install"
after_install = "app.setup.install.after_install"
```

**app/setup/install.py**
```py
def after_install():
	# run code after app installation
	pass
```

## Boot Session

After a successful login, the Desk is injected with a dictionary of global
values called `bootinfo`. The `bootinfo` is available as a global object in
Javascript as `frappe.boot`.

The `bootinfo` dict contains a lot of values including:

- System defaults
- Notification status
- Permissions
- User settings
- Language and timezone info

You can add global values that makes sense for your app via the `boot_session`
hook.

```py
# python module path
boot_session = "app.boot.boot_session"
```

The method is called with one argument `bootinfo`, on which you can directly
add/update values.

**app/boot.py**

```py
def boot_session(bootinfo):
	bootinfo.my_global_key = 'my_global_value'
```

Now, you can access the value anywhere in your client side code.
```js
console.log(frappe.boot.my_global_key)
```

## Notification configurations

The notification configuration hook is expected to return a Python dictionary.

	{
		"for_doctype": {
			"Issue": {"status":"Open"},
			"Customer Issue": {"status":"Open"},
		},
		"for_module_doctypes": {
			"ToDo": "To Do",
			"Event": "Calendar",
			"Comment": "Messages"
		},
		"for_module": {
			"To Do": "frappe.core.notifications.get_things_todo",
			"Calendar": "frappe.core.notifications.get_todays_events",
			"Messages": "frappe.core.notifications.get_unread_messages"
		}
	}


The above configuration has three parts,

1. `for_doctype` part of the above configuration marks any "Issue"
	or "Customer Issue" as unread if its status is Open
2. `for_module_doctypes` maps doctypes to module's unread count.
3. `for_module` maps modules to functions to obtain its unread count. The
   functions are called without any argument.


## Customizing Web Forms

> Introduced in Version 13

In order to modify the client-side validations and stylesheets for existing web forms for a given DocType in the system, you can use the following hooks:

1. `webform_include_js`
1. `webform_include_css`

Eg,

	webform_include_js = {
		"Issue": "public/js/issue.js",
		"Address": "public/js/address.js"
	}

## Configuring Reports

In the report view, you can force filters as per doctype using `dump_report_map`
hook. The hook should be a dotted path to a Python function which will be called
without any arguments. Example of output of this function is below.


	"Warehouse": {
		"columns": ["name"],
		"conditions": ["docstatus < 2"],
		"order_by": "name"
	}

Here, for a report with Warehouse doctype, would include only those records that
are not cancelled (docstatus < 2) and will be ordered by name.

## Modifying Website Context

Context used in website pages can be modified by adding
a `update_website_context` hook. This hook should be a dotted path to a function
which will be called with a context (dictionary) argument.

## Customizing Email footer

By default, for every email, a footer with content, "Sent via Frappe" is sent.
You can customize this globally by adding a `mail_footer` hook. The hook should
be a dotted path to a variable.

## Session Creation Hook

You can attach custom logic to the event of a successful login using
`on_session_creation` hook. The hook should be a dotted path to a Python
function that takes login\_manager as an argument.

Eg,

	def on_session_creation(login_manager):
		"""make feed"""
		if frappe.session['user'] != 'Guest':
			# log to timesheet
			pass

## Website Clear Cache

If you cache values in your views, the `website_clear_cache` allows you to hook
methods that invalidate your caches when Frappe tries to clear cache for all
website related pages.

### Document hooks

#### Permissions

#### Query Permissions
You can customize how permissions are resolved for a DocType by hooking custom
permission match conditions using the `permission_query_conditions` hook. This
match condition is expected to be fragment for a where clause in an sql query.
Structure for this hook is as follows.


	permission_query_conditions = {
		"{doctype}": "{dotted.path.to.function}",
	}

The output of the function should be a string with a match condition.
Example of such a function,


	def get_permission_query_conditions():
		return "(tabevent.event_type='public' or tabevent.owner='{user}'".format(user=frappe.session.user)

The above function returns a fragment that permits an event to listed if it's
public or owned by the current user.

#### Document permissions
You can hook to `doc.has_permission` for any DocType and add special permission
checking logic using the `has_permission` hook. Structure for this hook is,

	has_permission = {
		"{doctype}": "{dotted.path.to.function}",
	}

The function will be passed the concerned document as an argument. It should
True or a falsy value after running the required logic.

For Example,

	def has_permission(doc):
		if doc.event_type=="Public" or doc.owner==frappe.session.user:
			return True

The above function permits an event if it's public or owned by the current user.

#### CRUD Events

You can hook to various CRUD events of any doctype, the syntax for such a hook
is as follows,

	doc_events = {
		"{doctype}": {
			"{event}": "{dotted.path.to.function}",
		}
	}

To hook to events of all doctypes, you can use the follwing syntax also,

	 doc_events = {
	 	"*": {
	 		"on_update": "{dotted.path.to.function}",
		}
	 }

The hook function will be passed the doc in concern as the only argument.

##### List of events

* `validate`
* `before_save`
* `autoname`
* `before_insert`
* `after_insert`
* `before_submit`
* `before_cancel`
* `before_update_after_submit`
* `on_update`
* `on_submit`
* `on_cancel`
* `on_update_after_submit`
* `on_change`
* `on_trash`
* `after_delete`


Eg,

	doc_events = {
		"Cab Request": {
			"after_insert": "topcab.schedule_cab",
		}
	}

### Scheduler Hooks

Scheduler hooks are methods that are run periodically in background. Structure for such a hook is,

	scheduler_events = {
		"{event_name}": [
			"{dotted.path.to.function}"
		],
	}

#### Events

* `daily`
* `daily_long`
* `weekly`
* `weekly_long`
* `monthly`
* `monthly_long`
* `hourly`
* `all`
* `cron`

The scheduler events require RQ and redis (or a supported and
configured broker) to be running. The events with suffix '\_long' are for long
jobs. The `all` event is triggered everytime (as per the RQ interval).

Example,

	scheduler_events = {
		"daily": [
			"erpnext.accounts.doctype.sales_invoice.sales_invoice.manage_recurring_invoices"
		],
		"daily_long": [
			"erpnext.setup.doctype.backup_manager.backup_manager.take_backups_daily"
		],
		"cron": {
			"15 18 * * *": [
				"frappe.twofactor.delete_all_barcodes_for_users"
			],
			"*/6 * * * *": [
				"frappe.utils.error.collect_error_snapshots"
			],
			"annual": [
				"frappe.utils.error.collect_error_snapshots"
			]
		}
	}

The asterisk (*) operator specifies all possible values for a field. For example, an asterisk in the hour time field would be equivalent to every hour or an asterisk in the month field would be equivalent to every month.

### Override Whitelisted Methods

Override whitelisted Python functions (server-side functions that are available to the client-side) using the `override_whitelisted_methods` hook.

An overriden function will be triggered during the following events:

* direct API call
* document mapping

Example,

	override_whitelisted_methods = {
		"{dotted.path.to.whitelisted.frappe.function}": "{dotted.path.to.whitelisted.custom.function}"
	}

### Override DocType Class

You can override the class for standard doctypes by using the
`override_doctype_class` hook.

```py
override_doctype_class = {
	'ToDo': 'app.overrides.todo.CustomToDo'
}
```

**app/overrides/todo.py**
```py
from frappe.desk.doctype.todo.todo import ToDo

class CustomToDo(ToDo):
	def on_update(self):
		self.my_custom_code()
		super(ToDo, self).on_update()

	def my_custom_code(self):
		pass
```

> It is recommended that you extend the standard class of the doctype, otherwise
> you will have to implement all of the core functionality.

### Jinja Customization

Fetch custom methods and filters that are to be available globally in jinja environment.

#### Options

* `methods`
* `filters`

Example,

	jenv = {
		"methods": [
			"method_name:dotted.path.to.method_definition"
		],
		"filters": [
			"filter_name:dotted.path.to.filter_function"
		]
	}

### Exempt Doctypes

Exempt documents of a specific DocType from being automatically cancelled on the cancellation of any linked documents.

Example,

	auto_cancel_exempted_doctypes = ["Payment Entry"]

In the above example, if any document that is linked with Payment Entry is cancelled, the system will skip the auto-cancellation of the linked Payment Entry document.

### Form Timeline

Frappe forms show some predefined timeline contents like form views,
data changes, communications related to the current record, etc.

Apart from these standard contents, there might arise a situation where you need
to add custom timeline content. This can be done via `additional_timeline_content` hook.

**Usage:**

```Python
additional_timeline_content: {
	'*': ['path.to.the.method_which_returns_timeline_content'] # show in each DocType's timeline
	'ToDo': ['path.to.the.method_which_returns_timeline_content'] # only show in ToDo's timeline
}
```

```Python
def method_which_returns_timeline_content(doctype, docname):
	# this method should return a list of dicts
	# example
	return [{
		'creation': '22-05-2020 18:00:00' # this will be used to sort the content in the timeline
		'template': 'custom_timeline_template' # this Jinja template will be rendered in the timeline
		'template_data': {'key': 'value'} # this data will be passed to the template.
	}]
```
