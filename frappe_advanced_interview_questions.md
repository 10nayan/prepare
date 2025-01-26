# 100 Advanced Frappe Framework Interview Questions and Answers

## Frappe Basics and Architecture

1. **Q: What is Frappe Framework and what are its key features?**
   - A: Frappe is a full-stack web application framework built on Python and JavaScript. Key features include:
   - Meta-data driven architecture
   - Built-in ORM
   - Role-based permissions
   - REST API
   - Real-time updates
   - Extensible architecture
   - Document versioning
   - Background jobs
   - Built-in web forms
   - Multi-tenancy support

2. **Q: How does Frappe's architecture work?**
   ```python
   # Example of Frappe's MVC-like architecture
   # Model (DocType)
   class Customer(Document):
       def validate(self):
           self.validate_credit_limit()
   
   # Controller (Python API)
   @frappe.whitelist()
   def get_customer_details(customer_id):
       return frappe.get_doc("Customer", customer_id)
   
   # View (Form/List)
   # customer.js
   frappe.ui.form.on('Customer', {
       refresh: function(frm) {
           // Form rendering logic
       }
   })
   ```

3. **Q: Explain Frappe's database structure.**
   - A: Frappe uses MariaDB/MySQL with key components:
   ```sql
   -- Main DocType table
   CREATE TABLE `tabCustomer` (
       `name` varchar(140) PRIMARY KEY,
       `creation` datetime,
       `modified` datetime,
       `modified_by` varchar(140),
       `owner` varchar(140),
       `docstatus` int(1) DEFAULT 0,
       -- Custom fields follow
   );
   
   -- Single DocType
   CREATE TABLE `tabSingles` (
       `doctype` varchar(140),
       `field` varchar(140),
       `value` text
   );
   ```

4. **Q: How do you implement custom apps in Frappe?**
   ```bash
   # Creating new app
   bench new-app my_custom_app
   
   # App structure
   my_custom_app/
   ├── MANIFEST.in
   ├── README.md
   ├── my_custom_app/
   │   ├── __init__.py
   │   ├── hooks.py
   │   ├── modules.txt
   │   ├── patches.txt
   │   └── templates/
   └── setup.py
   ```

5. **Q: How do you handle hooks in Frappe?**
   ```python
   # hooks.py
   app_name = "my_custom_app"
   app_title = "My Custom App"
   
   doc_events = {
       "Sales Order": {
           "on_submit": "my_custom_app.events.so_on_submit",
           "on_cancel": "my_custom_app.events.so_on_cancel"
       }
   }
   
   scheduler_events = {
       "daily": [
           "my_custom_app.tasks.daily_cleanup"
       ]
   }
   ```

## DocTypes and Forms

6. **Q: How do you create custom DocTypes?**
   ```json
   {
       "name": "Custom DocType",
       "doctype": "DocType",
       "module": "My Custom App",
       "custom": 1,
       "fields": [
           {
               "fieldname": "title",
               "fieldtype": "Data",
               "label": "Title",
               "reqd": 1
           }
       ],
       "permissions": [
           {
               "role": "System Manager",
               "read": 1,
               "write": 1,
               "create": 1,
               "delete": 1
           }
       ]
   }
   ```

7. **Q: How do you implement custom form validations?**
   ```python
   # Python controller
   class CustomDoc(Document):
       def validate(self):
           if self.end_date < self.start_date:
               frappe.throw("End date cannot be before start date")
   
   # Client-side validation
   frappe.ui.form.on('Custom Doc', {
       validate: function(frm) {
           if (frm.doc.amount < 0) {
               frappe.throw(__("Amount cannot be negative"));
           }
       }
   });
   ```

8. **Q: How do you implement custom fields?**
   ```python
   # Adding custom field programmatically
   def create_custom_field():
       custom_field = frappe.get_doc({
           "doctype": "Custom Field",
           "dt": "Customer",
           "fieldname": "custom_field",
           "fieldtype": "Data",
           "label": "Custom Field",
           "insert_after": "customer_name"
       }).insert()
   ```

9. **Q: How do you implement child tables?**
   ```json
   {
       "name": "Parent DocType",
       "doctype": "DocType",
       "fields": [
           {
               "fieldname": "items",
               "fieldtype": "Table",
               "label": "Items",
               "options": "Child DocType"
           }
       ]
   }
   ```

10. **Q: How do you implement form scripts?**
    ```javascript
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            frm.add_custom_button(__('Custom Action'), function() {
                // Custom action logic
            });
        },
        
        validate: function(frm) {
            // Validation logic
        },
        
        before_save: function(frm) {
            // Pre-save logic
        }
    });
    ```

## Server Scripts and API

11. **Q: How do you implement server-side scripts?**
    ```python
    # Server script for custom validation
    def validate(doc, method):
        if doc.status == "Completed" and not doc.completion_date:
            frappe.throw("Completion date is mandatory")
    
    # Custom API method
    @frappe.whitelist()
    def custom_method(param1, param2):
        # Method logic
        return {"status": "success"}
    ```

12. **Q: How do you implement custom REST APIs?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def get_data():
        return frappe.get_list("DocType",
            fields=["name", "creation"],
            filters={"published": 1}
        )
    
    # Custom API endpoint
    @frappe.whitelist()
    def update_status(docname, status):
        doc = frappe.get_doc("DocType", docname)
        doc.status = status
        doc.save()
        return doc
    ```

13. **Q: How do you implement background jobs?**
    ```python
    # Queuing a background job
    def enqueue_job():
        frappe.enqueue(
            'my_custom_app.tasks.long_running_task',
            queue='long',
            timeout=300,
            doc_name=self.name
        )
    
    # Background job implementation
    def long_running_task(doc_name):
        # Long running process
        frappe.publish_realtime(
            'task_progress',
            {'progress': 50, 'doc_name': doc_name}
        )
    ```

14. **Q: How do you implement custom commands?**
    ```python
    # Custom bench command
    import click
    from frappe.commands import pass_context
    
    @click.command('custom-command')
    @pass_context
    def custom_command(context):
        """Custom command description"""
        frappe.init(site=context.sites[0])
        frappe.connect()
        # Command logic
        frappe.destroy()
    
    commands = [custom_command]
    ```

15. **Q: How do you implement document events?**
    ```python
    # Document event handlers
    def on_update(doc, method):
        frappe.msgprint(f"Document {doc.name} updated")
    
    def on_trash(doc, method):
        # Cleanup related records
        frappe.delete_doc("Related DocType", 
                         filters={"parent": doc.name})
    ```

## Client Scripts and Events

16. **Q: How do you implement client-side scripting?**
    ```javascript
    // Client script
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            // Add custom button
            frm.add_custom_button(__('Process'), function() {
                process_document(frm);
            });
        },
        
        custom_field: function(frm) {
            // Field change handler
            calculate_totals(frm);
        }
    });
    
    function process_document(frm) {
        frappe.call({
            method: 'my_custom_app.api.process_doc',
            args: {
                docname: frm.doc.name
            },
            callback: function(r) {
                frappe.msgprint(r.message);
            }
        });
    }
    ```

17. **Q: How do you implement custom dialogs?**
    ```javascript
    function show_custom_dialog() {
        let d = new frappe.ui.Dialog({
            title: 'Enter details',
            fields: [
                {
                    label: 'First Name',
                    fieldname: 'first_name',
                    fieldtype: 'Data'
                },
                {
                    label: 'Last Name',
                    fieldname: 'last_name',
                    fieldtype: 'Data'
                }
            ],
            primary_action_label: 'Submit',
            primary_action(values) {
                // Handle submission
                d.hide();
            }
        });
        
        d.show();
    }
    ```

18. **Q: How do you implement custom list views?**
    ```javascript
    frappe.listview_settings['DocType'] = {
        add_fields: ['status', 'priority'],
        get_indicator: function(doc) {
            if (doc.status === "Completed") {
                return [__("Completed"), "green", "status,=,Completed"];
            }
        },
        onload: function(listview) {
            // Custom list view logic
        }
    };
    ```

19. **Q: How do you implement custom workspaces?**
    ```python
    # workspace.py
    def get_data():
        return {
            'cards': [
                {
                    'label': 'Sales',
                    'items': [
                        {
                            'type': 'doctype',
                            'name': 'Sales Order',
                            'label': 'Sales Orders'
                        }
                    ]
                }
            ],
            'charts': [
                {
                    'name': 'Sales Analytics',
                    'chart_name': 'Sales Trends'
                }
            ]
        }
    ```

20. **Q: How do you implement custom reports?**
    ```python
    from frappe import _
    
    def execute(filters=None):
        columns = get_columns()
        data = get_data(filters)
        return columns, data
    
    def get_columns():
        return [
            {
                "fieldname": "name",
                "label": _("Name"),
                "fieldtype": "Link",
                "options": "DocType",
                "width": 140
            }
        ]
    ```

## Reports and Queries

21. **Q: How do you implement custom query reports?**
    ```python
    # query_report.py
    def get_data(filters):
        return frappe.db.sql("""
            SELECT 
                name, customer_name, grand_total
            FROM 
                `tabSales Invoice`
            WHERE 
                posting_date BETWEEN %(from_date)s AND %(to_date)s
        """, filters, as_dict=1)
    ```

22. **Q: How do you implement script reports?**
    ```python
    # script_report.py
    class CustomReport(object):
        def __init__(self, filters=None):
            self.filters = filters
        
        def run(self):
            columns = self.get_columns()
            data = self.get_data()
            return columns, data
        
        def get_columns(self):
            return [
                {"label": "ID", "fieldname": "id", "width": 100},
                {"label": "Name", "fieldname": "name", "width": 200}
            ]
    ```

23. **Q: How do you implement custom filters?**
    ```javascript
    // Custom filter
    frappe.query_reports["Custom Report"] = {
        "filters": [
            {
                "fieldname": "from_date",
                "label": __("From Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.add_months(
                    frappe.datetime.get_today(), -1
                ),
                "reqd": 1
            },
            {
                "fieldname": "to_date",
                "label": __("To Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.get_today(),
                "reqd": 1
            }
        ]
    };
    ```

24. **Q: How do you implement custom queries?**
    ```python
    # Custom database queries
    def get_custom_data():
        return frappe.db.sql("""
            SELECT 
                t1.name, 
                t1.customer_name,
                t2.item_code,
                t2.qty
            FROM 
                `tabSales Order` t1
                LEFT JOIN `tabSales Order Item` t2 
                ON t1.name = t2.parent
            WHERE 
                t1.docstatus = 1
        """, as_dict=True)
    ```

25. **Q: How do you implement report charts?**
    ```python
    def get_chart_data(data):
        labels = []
        values = []
        
        for d in data:
            labels.append(d.get("month"))
            values.append(d.get("amount"))
        
        return {
            "data": {
                "labels": labels,
                "datasets": [
                    {
                        "name": "Monthly Sales",
                        "values": values
                    }
                ]
            },
            "type": "line"
        }
    ```

## Customization and Extensions

26. **Q: How do you implement custom apps?**
    ```python
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    app_include_css = [
        "/assets/custom_app/css/custom.css"
    ]
    ```

27. **Q: How do you implement custom fields programmatically?**
    ```python
    def create_custom_fields():
        custom_fields = {
            "Customer": [
                {
                    "fieldname": "custom_field",
                    "label": "Custom Field",
                    "fieldtype": "Data",
                    "insert_after": "customer_name"
                }
            ]
        }
        
        for doctype, fields in custom_fields.items():
            for field in fields:
                if not frappe.db.exists(
                    "Custom Field", 
                    {"dt": doctype, "fieldname": field["fieldname"]}
                ):
                    frappe.get_doc({
                        "doctype": "Custom Field",
                        "dt": doctype,
                        **field
                    }).insert()
    ```

28. **Q: How do you implement custom roles and permissions?**
    ```python
    def create_custom_role():
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Role"
            role.desk_access = 1
            role.insert()
    
    def set_custom_permissions():
        # Custom DocPerm
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.read = 1
        docperm.write = 1
        docperm.create = 1
        docperm.submit = 0
        docperm.cancel = 0
        docperm.delete = 0
        docperm.insert()
    ```

29. **Q: How do you implement custom print formats?**
    ```html
    <!-- custom_format.html -->
    <div class="print-format">
        <div class="row">
            <div class="col-xs-6">
                <div class="row">
                    <div class="col-xs-5">
                        <img src="{{ letter_head }}">
                    </div>
                </div>
            </div>
        </div>
        
        <div class="row">
            <div class="col-xs-12">
                <div class="row">
                    <div class="col-xs-6">
                        <strong>{{ doc.customer_name }}</strong><br>
                        {{ doc.address_display }}
                    </div>
                </div>
            </div>
        </div>
    </div>
    ```

30. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.new_doc("Workflow")
        workflow.workflow_name = "Custom Workflow"
        workflow.document_type = "Custom DocType"
        workflow.workflow_state_field = "workflow_state"
        
        # States
        workflow.states = [
            {
                "state": "Draft",
                "doc_status": 0,
                "allow_edit": "Custom Role"
            },
            {
                "state": "Submitted",
                "doc_status": 1,
                "allow_edit": "System Manager"
            }
        ]
        
        # Transitions
        workflow.transitions = [
            {
                "state": "Draft",
                "action": "Submit",
                "next_state": "Submitted",
                "allowed": "Custom Role"
            }
        ]
        
        workflow.insert()
    ```

## Workflow and Automation

31. **Q: How do you implement custom workflows with multiple states?**
    ```python
    def setup_workflow():
        workflow = {
            "name": "Custom Process",
            "document_type": "Custom DocType",
            "states": [
                {
                    "state": "Draft",
                    "style": "Primary",
                    "doc_status": 0
                },
                {
                    "state": "Pending Approval",
                    "style": "Warning",
                    "doc_status": 0
                },
                {
                    "state": "Approved",
                    "style": "Success",
                    "doc_status": 1
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Custom User"
                },
                {
                    "state": "Pending Approval",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "Custom Approver"
                }
            ]
        }
        
        if not frappe.db.exists("Workflow", workflow["name"]):
            doc = frappe.get_doc({"doctype": "Workflow", **workflow})
            doc.insert()
    ```

32. **Q: How do you implement scheduled tasks?**
    ```python
    # hooks.py
    scheduler_events = {
        "daily": [
            "custom_app.tasks.daily_cleanup"
        ],
        "hourly": [
            "custom_app.tasks.send_reminders"
        ],
        "weekly": [
            "custom_app.tasks.weekly_report"
        ]
    }
    
    # tasks.py
    def daily_cleanup():
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    
    def send_reminders():
        # Send reminder emails
        pass
    ```

33. **Q: How do you implement document event handlers?**
    ```python
    # hooks.py
    doc_events = {
        "Sales Order": {
            "on_submit": "custom_app.events.so_on_submit",
            "on_cancel": "custom_app.events.so_on_cancel",
            "after_insert": "custom_app.events.so_after_insert"
        }
    }
    
    # events.py
    def so_on_submit(doc, method):
        # Create related documents
        create_project(doc)
        notify_departments(doc)
    
    def notify_departments(doc):
        frappe.publish_realtime(
            'sales_order_submitted',
            {"sales_order": doc.name}
        )
    ```

34. **Q: How do you implement custom notifications?**
    ```python
    def setup_notification():
        notification = frappe.new_doc("Notification")
        notification.name = "Custom Alert"
        notification.subject = "New {{ doc.doctype }} {{ doc.name }}"
        notification.document_type = "Custom DocType"
        notification.event = "Submit"
        notification.channel = "Email"
        notification.message = """
            Dear {{ doc.customer_name }},
            Your {{ doc.doctype }} {{ doc.name }} has been submitted.
        """
        notification.insert()
    ```

35. **Q: How do you implement email alerts?**
    ```python
    def send_custom_email(doc):
        frappe.sendmail(
            recipients=[doc.email],
            subject=f"Document {doc.name} Status Update",
            template="custom_email",
            args={
                "name": doc.name,
                "status": doc.status,
                "comments": doc.comments
            },
            header=["Status Update", "green"]
        )
    
    # custom_email.html
    <h3>Status Update</h3>
    <p>Document {{ name }} status has been updated to {{ status }}</p>
    {% if comments %}
    <p>Comments: {{ comments }}</p>
    {% endif %}
    ```

## Security and Permissions

36. **Q: How do you implement role-based access control?**
    ```python
    def setup_roles_and_permissions():
        # Create custom role
        if not frappe.db.exists("Role", "Custom Manager"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Manager"
            role.desk_access = 1
            role.insert()
        
        # Set permissions
        custom_manager_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        add_permission(
            "Custom DocType",
            "Custom Manager",
            custom_manager_perms
        )
    ```

37. **Q: How do you implement custom authentication?**
    ```python
    from frappe.auth import LoginManager
    
    class CustomLoginManager(LoginManager):
        def validate_user(self):
            # Custom validation logic
            super().validate_user()
            
            if not self.user_in_approved_list():
                frappe.throw("User not approved")
        
        def user_in_approved_list(self):
            return frappe.db.exists(
                "Approved Users",
                {"user": self.user}
            )
    ```

38. **Q: How do you implement field-level permissions?**
    ```python
    def setup_field_permissions():
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.permlevel = 1
        docperm.read = 1
        docperm.write = 0
        
        # Field permission in DocType
        custom_field = {
            "fieldname": "sensitive_data",
            "permlevel": 1
        }
        
        update_field_permission(
            "Custom DocType",
            custom_field
        )
    ```

39. **Q: How do you implement API authentication?**
    ```python
    @frappe.whitelist()
    def custom_api():
        # Check if user has permission
        if not frappe.has_permission("Custom DocType"):
            frappe.throw("Not permitted")
        
        # API token validation
        api_token = frappe.get_request_header("Authorization")
        if not validate_api_token(api_token):
            frappe.throw("Invalid API token")
        
        return "API response"
    
    def validate_api_token(token):
        return frappe.db.exists(
            "API Token",
            {"token": token, "enabled": 1}
        )
    ```

40. **Q: How do you implement user permissions?**
    ```python
    def setup_user_permissions():
        # Add user permission
        user_permission = frappe.new_doc("User Permission")
        user_permission.user = "user@example.com"
        user_permission.allow = "Custom DocType"
        user_permission.for_value = "CUST-001"
        user_permission.apply_to_all_doctypes = 1
        user_permission.insert()
        
        # Check user permission
        def validate_user_permission(doc, method):
            if not frappe.has_permission(
                doc.doctype,
                "write",
                doc
            ):
                frappe.throw("Not permitted")
    ```

## Deployment and Performance

41. **Q: How do you implement custom deployment configurations?**
    ```python
    # config.py
    from frappe.utils import cstr, cint
    
    def get_site_config():
        return {
            'maintenance_mode': 0,
            'pause_scheduler': 0,
            'developer_mode': 0,
            'disable_website_cache': 0,
            'rate_limit': {
                'limit': 100,
                'window': 3600
            }
        }
    ```

42. **Q: How do you implement caching strategies?**
    ```python
    from frappe.utils.caching import redis_cache
    
    @redis_cache
    def get_cached_data(key):
        # Expensive operation
        result = frappe.db.sql("""
            SELECT * FROM `tabCustom DocType`
            WHERE status = 'Active'
        """, as_dict=True)
        return result
    
    def clear_cache():
        frappe.cache().delete_key('cached_data')
    ```

43. **Q: How do you implement database optimization?**
    ```python
    def optimize_database():
        # Add indexes
        frappe.db.sql("""
            CREATE INDEX IF NOT EXISTS idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize tables
        frappe.db.sql("""
            OPTIMIZE TABLE `tabCustom DocType`
        """)
        
        # Clear unused data
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

44. **Q: How do you implement load balancing?**
    ```python
    # nginx configuration
    def get_nginx_config():
        return """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
        """
    ```

45. **Q: How do you implement monitoring and logging?**
    ```python
    import logging
    
    def setup_logging():
        # Custom logger
        logger = logging.getLogger("custom_app")
        logger.setLevel(logging.DEBUG)
        
        # File handler
        fh = logging.FileHandler("custom_app.log")
        fh.setLevel(logging.DEBUG)
        
        # Formatter
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        fh.setFormatter(formatter)
        logger.addHandler(fh)
        
        return logger
    
    # Usage
    logger = setup_logging()
    logger.info("Custom operation completed")
    ```

## Integration and Development

46. **Q: How do you implement external API integration?**
    ```python
    import requests
    
    class ExternalAPIIntegration:
        def __init__(self):
            self.api_key = frappe.conf.get("external_api_key")
            self.base_url = "https://api.external.com/v1"
        
        def make_request(self, endpoint, method="GET", data=None):
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            response = requests.request(
                method,
                f"{self.base_url}/{endpoint}",
                headers=headers,
                json=data
            )
            
            response.raise_for_status()
            return response.json()
    ```

47. **Q: How do you implement webhooks?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def webhook_handler():
        if not validate_webhook_signature():
            frappe.throw("Invalid webhook signature")
        
        data = frappe.request.get_json()
        
        # Process webhook data
        process_webhook_data(data)
        
        return "Success"
    
    def validate_webhook_signature():
        signature = frappe.get_request_header("X-Webhook-Signature")
        # Validate signature
        return True
    ```

48. **Q: How do you implement custom REST endpoints?**
    ```python
    @frappe.whitelist()
    def custom_endpoint():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

49. **Q: How do you implement file handling?**
    ```python
    def handle_file_upload():
        file_doc = frappe.get_doc({
            "doctype": "File",
            "attached_to_doctype": "Custom DocType",
            "attached_to_name": "DOC-001",
            "attached_to_field": "attachment",
            "file_url": "",
            "file_name": "document.pdf",
            "is_private": 1,
            "content": frappe.request.files['file']
        }).insert()
        
        return file_doc
    ```

50. **Q: How do you implement custom apps and modules?**
    ```python
    # modules.txt
    Custom App
    Core
    Integration
    
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    ```

## ERPNext Integration

51. **Q: How do you extend ERPNext DocTypes?**
    ```python
    # custom_sales_order.py
    from erpnext.selling.doctype.sales_order.sales_order import SalesOrder
    
    class CustomSalesOrder(SalesOrder):
        def validate(self):
            super().validate()
            self.validate_custom_fields()
        
        def validate_custom_fields(self):
            if self.custom_field < 0:
                frappe.throw("Custom field cannot be negative")
    ```

52. **Q: How do you implement custom pricing rules?**
    ```python
    def get_custom_price(item_code, customer):
        pricing_rule = frappe.get_doc({
            "doctype": "Pricing Rule",
            "title": f"Custom Price for {customer}",
            "apply_on": "Item Code",
            "items": [{
                "item_code": item_code
            }],
            "selling": 1,
            "rate_or_discount": "Rate",
            "rate": 100
        })
        return pricing_rule
    ```

53. **Q: How do you implement custom workflows in ERPNext?**
    ```python
    def create_custom_workflow():
        workflow = {
            "name": "Custom SO Workflow",
            "document_type": "Sales Order",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Sales User"
                },
                {
                    "state": "Pending Approval",
                    "doc_status": 0,
                    "allow_edit": "Sales Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Sales User"
                }
            ]
        }
        return workflow
    ```

54. **Q: How do you implement custom reports in ERPNext?**
    ```python
    def get_custom_report():
        return {
            "columns": [
                {
                    "fieldname": "customer",
                    "label": "Customer",
                    "fieldtype": "Link",
                    "options": "Customer",
                    "width": 180
                },
                {
                    "fieldname": "total_sales",
                    "label": "Total Sales",
                    "fieldtype": "Currency",
                    "width": 140
                }
            ],
            "filters": [
                {
                    "fieldname": "from_date",
                    "label": "From Date",
                    "fieldtype": "Date",
                    "default": frappe.utils.add_months(
                        frappe.utils.nowdate(), -1
                    )
                }
            ]
        }
    ```

55. **Q: How do you implement custom item variants?**
    ```python
    def create_item_variant():
        template = frappe.get_doc("Item", "ITEM-TEMP-001")
        variant = frappe.get_doc({
            "doctype": "Item",
            "variant_of": template.name,
            "item_code": "ITEM-VAR-001",
            "item_name": "Custom Variant",
            "item_group": template.item_group,
            "stock_uom": template.stock_uom,
            "attributes": [
                {
                    "attribute": "Size",
                    "attribute_value": "Large"
                }
            ]
        }).insert()
        return variant
    ```

## Advanced Development

56. **Q: How do you implement custom web pages?**
    ```python
    # hooks.py
    website_route_rules = [
        {"from_route": "/custom", "to_route": "custom/index"}
    ]
    
    # custom/index.html
    {% extends "templates/web.html" %}
    
    {% block page_content %}
    <div class="custom-page">
        <h1>{{ title }}</h1>
        {{ content }}
    </div>
    {% endblock %}
    
    # custom/index.py
    def get_context(context):
        context.title = "Custom Page"
        context.content = frappe.get_doc(
            "Custom Content", 
            "CONTENT-001"
        ).content
    ```

57. **Q: How do you implement custom portals?**
    ```python
    # hooks.py
    portal_menu_items = [
        {
            "title": "Custom Portal",
            "route": "/custom-portal",
            "reference_doctype": "Custom DocType",
            "role": "Customer"
        }
    ]
    
    # custom_portal.js
    frappe.pages['custom-portal'].on_page_load = function(wrapper) {
        var page = frappe.ui.make_app_page({
            parent: wrapper,
            title: 'Custom Portal',
            single_column: true
        });
        
        page.add_inner_button(__('New'), function() {
            frappe.new_doc('Custom DocType');
        });
    }
    ```

58. **Q: How do you implement custom dashboards?**
    ```python
    def get_dashboard_data(data):
        return {
            'fieldname': 'custom_doctype',
            'transactions': [
                {
                    'label': _('Related Documents'),
                    'items': ['Sales Order', 'Purchase Order']
                }
            ],
            'charts': [
                {
                    'name': 'Custom Chart',
                    'chart_name': 'Monthly Trends',
                    'chart_type': 'Line'
                }
            ]
        }
    ```

59. **Q: How do you implement custom scripts?**
    ```python
    # Server Script
    doc = frappe.get_doc({
        "doctype": "Server Script",
        "name": "Custom Server Script",
        "script_type": "DocType Event",
        "doc_type": "Custom DocType",
        "event": "Before Save",
        "script": """
    if doc.status == "Completed" and not doc.completion_date:
        doc.completion_date = frappe.utils.nowdate()
        """
    })
    
    # Client Script
    doc = frappe.get_doc({
        "doctype": "Client Script",
        "name": "Custom Client Script",
        "dt": "Custom DocType",
        "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
        """
    })
    ```

60. **Q: How do you implement custom APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    
    def process_request(params):
        # Custom processing logic
        return {
            "processed": True,
            "params": params
        }
    ```

## Advanced Features

61. **Q: How do you implement custom email templates?**
    ```python
    def create_email_template():
        template = frappe.get_doc({
            "doctype": "Email Template",
            "name": "Custom Template",
            "subject": "{{ doc.name }} - Status Update",
            "response": """
                Dear {{ doc.customer_name }},
                
                Your {{ doc.doctype }} ({{ doc.name }}) 
                has been updated to {{ doc.status }}.
                
                Regards,
                {{ frappe.defaults.get_global_default('company') }}
            """
        }).insert()
        return template
    ```

62. **Q: How do you implement custom print formats?**
    ```python
    def create_print_format():
        format_doc = frappe.get_doc({
            "doctype": "Print Format",
            "name": "Custom Format",
            "doc_type": "Custom DocType",
            "standard": "No",
            "print_format_type": "Jinja",
            "html": """
                <div class="print-heading">
                    <h2>{{ doc.name }}</h2>
                </div>
                
                <div class="row">
                    <div class="col-xs-6">
                        <div class="row">
                            <div class="col-xs-5">
                                <label>Customer</label>
                            </div>
                            <div class="col-xs-7">
                                {{ doc.customer_name }}
                            </div>
                        </div>
                    </div>
                </div>
            """
        }).insert()
        return format_doc
    ```

63. **Q: How do you implement custom web forms?**
    ```python
    def create_web_form():
        web_form = frappe.get_doc({
            "doctype": "Web Form",
            "title": "Custom Form",
            "route": "custom-form",
            "doc_type": "Custom DocType",
            "web_form_fields": [
                {
                    "fieldname": "customer_name",
                    "fieldtype": "Data",
                    "label": "Customer Name",
                    "reqd": 1
                },
                {
                    "fieldname": "email",
                    "fieldtype": "Data",
                    "label": "Email",
                    "reqd": 1,
                    "options": "Email"
                }
            ]
        }).insert()
        return web_form
    ```

64. **Q: How do you implement custom roles and permissions?**
    ```python
    def setup_custom_role():
        # Create role
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.get_doc({
                "doctype": "Role",
                "role_name": "Custom Role",
                "desk_access": 1
            }).insert()
        
        # Set permissions
        custom_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        if not frappe.db.exists(
            "Custom DocPerm",
            {
                "parent": "Custom DocType",
                "role": "Custom Role"
            }
        ):
            perm = frappe.get_doc({
                "doctype": "Custom DocPerm",
                "parent": "Custom DocType",
                "parenttype": "DocType",
                "parentfield": "permissions",
                "role": "Custom Role",
                **custom_perms
            }).insert()
    ```

65. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.get_doc({
            "doctype": "Workflow",
            "workflow_name": "Custom Process",
            "document_type": "Custom DocType",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Custom Role"
                },
                {
                    "state": "Pending",
                    "doc_status": 0,
                    "allow_edit": "System Manager"
                },
                {
                    "state": "Approved",
                    "doc_status": 1,
                    "allow_edit": "System Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit",
                    "next_state": "Pending",
                    "allowed": "Custom Role"
                },
                {
                    "state": "Pending",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "System Manager"
                }
            ]
        }).insert()
        return workflow
    ```

## Testing and Debugging

66. **Q: How do you implement unit tests?**
    ```python
    import unittest
    
    class TestCustomDoc(unittest.TestCase):
        def setUp(self):
            # Create test data
            self.doc = frappe.get_doc({
                "doctype": "Custom DocType",
                "title": "Test Doc"
            }).insert()
        
        def tearDown(self):
            # Cleanup test data
            frappe.delete_doc(
                "Custom DocType",
                self.doc.name
            )
        
        def test_validation(self):
            self.doc.status = "Completed"
            self.assertRaises(
                frappe.ValidationError,
                self.doc.save
            )
    ```

67. **Q: How do you implement integration tests?**
    ```python
    class TestIntegration(unittest.TestCase):
        def test_api_integration(self):
            # Test API endpoint
            response = frappe.call(
                'custom_app.api.custom_endpoint',
                param1="test",
                param2="value"
            )
            
            self.assertEqual(
                response.get("status"),
                "success"
            )
        
        def test_workflow(self):
            # Test workflow transitions
            doc = create_test_doc()
            doc.submit()
            
            self.assertEqual(
                doc.workflow_state,
                "Pending Approval"
            )
    ```

68. **Q: How do you implement custom error handling?**
    ```python
    class CustomError(Exception):
        def __init__(self, message, error_code=None):
            super().__init__(message)
            self.error_code = error_code
    
    def custom_error_handler():
        try:
            # Risky operation
            result = process_data()
            
            if not result:
                raise CustomError(
                    "Processing failed",
                    "PROC001"
                )
            
            return result
            
        except CustomError as e:
            frappe.log_error(
                title="Custom Error",
                message=str(e)
            )
            frappe.throw(
                str(e),
                exc=e.__class__
            )
    ```

69. **Q: How do you implement debugging tools?**
    ```python
    def debug_document(doc):
        # Log document state
        frappe.logger().debug({
            "doctype": doc.doctype,
            "name": doc.name,
            "status": doc.status,
            "modified": doc.modified
        })
        
        # Track changes
        frappe.logger().debug({
            "changed_fields": doc.get_changed(),
            "user": frappe.session.user,
            "timestamp": frappe.utils.now()
        })
    ```

70. **Q: How do you implement performance profiling?**
    ```python
    import cProfile
    import pstats
    
    def profile_function():
        profiler = cProfile.Profile()
        profiler.enable()
        
        # Function to profile
        result = expensive_operation()
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats()
        
        return result
    ```

## Advanced Customization

71. **Q: How do you implement custom field types?**
    ```python
    class CustomField(frappe.model.document.Document):
        def validate(self):
            if self.fieldtype == "Custom":
                self.validate_custom_field()
        
        def validate_custom_field(self):
            # Custom validation logic
            if not self.options:
                frappe.throw(
                    "Options required for Custom field type"
                )
    ```

72. **Q: How do you implement custom controllers?**
    ```python
    class CustomController(frappe.model.document.Document):
        def validate(self):
            self.validate_custom_logic()
        
        def on_submit(self):
            self.create_related_docs()
        
        def validate_custom_logic(self):
            if self.end_date <= self.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
        
        def create_related_docs(self):
            # Create related documents
            self.create_project()
            self.create_tasks()
    ```

73. **Q: How do you implement custom doctypes?**
    ```python
    def create_custom_doctype():
        doctype = frappe.get_doc({
            "doctype": "DocType",
            "name": "Custom DocType",
            "module": "Custom App",
            "custom": 1,
            "fields": [
                {
                    "fieldname": "title",
                    "fieldtype": "Data",
                    "label": "Title",
                    "reqd": 1
                },
                {
                    "fieldname": "status",
                    "fieldtype": "Select",
                    "label": "Status",
                    "options": "Draft\nPending\nCompleted"
                }
            ],
            "permissions": [
                {
                    "role": "System Manager",
                    "read": 1,
                    "write": 1,
                    "create": 1,
                    "delete": 1
                }
            ]
        }).insert()
        return doctype
    ```

74. **Q: How do you implement custom scripts?**
    ```python
    def create_custom_scripts():
        # Server script
        server_script = frappe.get_doc({
            "doctype": "Server Script",
            "name": "Custom Server Script",
            "script_type": "DocType Event",
            "doc_type": "Custom DocType",
            "event": "Before Save",
            "script": """
    if doc.status == "Completed":
        doc.completion_date = frappe.utils.now()
            """
        }).insert()
        
        # Client script
        client_script = frappe.get_doc({
            "doctype": "Client Script",
            "name": "Custom Client Script",
            "dt": "Custom DocType",
            "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
            """
        }).insert()
    ```

75. **Q: How do you implement custom reports?**
    ```python
    def create_custom_report():
        report = frappe.get_doc({
            "doctype": "Report",
            "name": "Custom Report",
            "ref_doctype": "Custom DocType",
            "report_type": "Script Report",
            "is_standard": "No",
            "module": "Custom App",
            "query": """
    SELECT
        name,
        creation,
        modified,
        status
    FROM
        `tabCustom DocType`
    WHERE
        status = %(status)s
            """,
            "filters": [
                {
                    "fieldname": "status",
                    "label": "Status",
                    "fieldtype": "Select",
                    "options": "Draft\nPending\nCompleted",
                    "default": "Pending"
                }
            ]
        }).insert()
        return report
    ```

## Data Management

76. **Q: How do you implement data import/export?**
    ```python
    def import_data():
        from frappe.core.doctype.data_import.data_import import (
            import_file
        )
        
        # Import data
        import_file(
            doctype="Custom DocType",
            file_path="path/to/import.csv",
            import_type="Insert",
            submit_after_import=True
        )
    
    def export_data():
        from frappe.desk.query_report import export_query
        
        # Export data
        export_query(
            "Custom Report",
            "CSV",
            filters={"status": "Completed"}
        )
    ```

77. **Q: How do you implement data migrations?**
    ```python
    def migrate_data():
        # Create migration plan
        plan = frappe.get_doc({
            "doctype": "Data Migration Plan",
            "name": "Custom Migration",
            "module": "Custom App",
            "mappings": [
                {
                    "mapping": "Customer to Contact",
                    "remote_objectname": "Contact",
                    "page_length": 10
                }
            ]
        })
        
        # Run migration
        run = frappe.get_doc({
            "doctype": "Data Migration Run",
            "data_migration_plan": plan.name
        }).insert()
        
        run.run()
    ```

78. **Q: How do you implement data archival?**
    ```python
    def archive_data():
        # Archive old records
        frappe.db.sql("""
            INSERT INTO `tabArchived DocType`
            SELECT *
            FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
        
        # Delete archived records
        frappe.db.sql("""
            DELETE FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
    ```

79. **Q: How do you implement data validation?**
    ```python
    def validate_data():
        def validate_custom_doc(doc, method):
            # Validate required fields
            if not doc.title:
                frappe.throw("Title is required")
            
            # Validate business logic
            if doc.end_date <= doc.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
            
            # Validate uniqueness
            if frappe.db.exists(
                "Custom DocType",
                {
                    "title": doc.title,
                    "name": ("!=", doc.name)
                }
            ):
                frappe.throw("Title must be unique")
    ```

80. **Q: How do you implement data cleanup?**
    ```python
    def cleanup_data():
        # Delete temporary files
        frappe.db.sql("""
            DELETE FROM `tabFile`
            WHERE attached_to_doctype = 'Custom DocType'
            AND creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
        
        # Clear error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 7 DAY)
        """)
        
        # Remove orphaned records
        frappe.db.sql("""
            DELETE FROM `tabCustom Child Table`
            WHERE parent NOT IN (
                SELECT name
                FROM `tabCustom DocType`
            )
        """)
    ```

## System Administration

81. **Q: How do you implement system monitoring?**
    ```python
    def monitor_system():
        # Monitor database
        db_stats = frappe.db.sql("""
            SHOW STATUS
            WHERE Variable_name IN (
                'Threads_connected',
                'Questions',
                'Slow_queries'
            )
        """, as_dict=1)
        
        # Monitor background jobs
        jobs = frappe.db.sql("""
            SELECT status, COUNT(*) as count
            FROM `tabQueue Job`
            GROUP BY status
        """, as_dict=1)
        
        # Log statistics
        frappe.logger().info({
            "db_stats": db_stats,
            "background_jobs": jobs
        })
    ```

82. **Q: How do you implement backup and recovery?**
    ```python
    def manage_backups():
        from frappe.utils.backups import backup
        
        # Create backup
        backup_path = backup(
            with_files=True,
            backup_path_db="/path/to/db/backup",
            backup_path_files="/path/to/files/backup"
        )
        
        # Restore from backup
        def restore_backup():
            # Stop server
            os.system("bench stop")
            
            # Restore database
            os.system(
                f"mysql -u {db_name} < {backup_path}/database.sql"
            )
            
            # Restore files
            os.system(
                f"rsync -a {backup_path}/files/ ./sites/site1.local/public/files/"
            )
            
            # Start server
            os.system("bench start")
    ```

83. **Q: How do you implement system updates?**
    ```python
    def update_system():
        # Update apps
        os.system("bench update")
        
        # Update dependencies
        os.system("bench setup requirements")
        
        # Migrate database
        os.system("bench migrate")
        
        # Clear cache
        frappe.clear_cache()
        
        # Restart server
        os.system("bench restart")
    ```

84. **Q: How do you implement system maintenance?**
    ```python
    def maintain_system():
        # Cleanup temporary files
        cleanup_temporary_files()
        
        # Optimize database
        frappe.db.sql("OPTIMIZE TABLE `tabCustom DocType`")
        
        # Clear cache
        frappe.clear_cache()
        
        # Delete old error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

85. **Q: How do you implement system security?**
    ```python
    def secure_system():
        # Set secure configuration
        frappe.db.set_value(
            "System Settings",
            None,
            {
                "allow_login_using_mobile_number": 0,
                "allow_login_using_user_name": 0,
                "allow_guests_to_upload_files": 0,
                "force_user_to_reset_password": 1
            }
        )
        
        # Enable two-factor authentication
        frappe.db.set_value(
            "System Settings",
            None,
            "enable_two_factor_auth",
            1
        )
    ```

## Performance Optimization

86. **Q: How do you implement caching strategies?**
    ```python
    def implement_caching():
        # Redis cache
        @frappe.whitelist()
        def get_cached_data():
            cache_key = "custom_data"
            data = frappe.cache().get_value(cache_key)
            
            if not data:
                data = get_expensive_data()
                frappe.cache().set_value(
                    cache_key,
                    data,
                    expires_in_sec=3600
                )
            
            return data
    ```

87. **Q: How do you implement query optimization?**
    ```python
    def optimize_queries():
        # Use proper indexes
        frappe.db.sql("""
            CREATE INDEX idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize joins
        results = frappe.db.sql("""
            SELECT t1.name, t2.item_code
            FROM `tabCustom DocType` t1
            INNER JOIN `tabCustom Child Table` t2
            ON t1.name = t2.parent
            WHERE t1.status = 'Active'
        """, as_dict=1)
    ```

88. **Q: How do you implement background jobs?**
    ```python
    def manage_background_jobs():
        # Enqueue long-running job
        frappe.enqueue(
            'custom_app.tasks.process_data',
            queue='long',
            timeout=1500,
            doc_name="DOC-001"
        )
        
        # Monitor job status
        def get_job_status(job_id):
            return frappe.db.get_value(
                'Queue Job',
                {'job_id': job_id},
                'status'
            )
    ```

89. **Q: How do you implement load balancing?**
    ```python
    def configure_load_balancing():
        # Nginx configuration
        nginx_config = """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
            }
        }
        """
    ```

90. **Q: How do you implement performance monitoring?**
    ```python
    def monitor_performance():
        import time
        
        def log_slow_queries():
            start_time = time.time()
            
            # Execute query
            result = frappe.db.sql(
                "SELECT * FROM `tabCustom DocType`"
            )
            
            execution_time = time.time() - start_time
            
            if execution_time > 1.0:  # Log if > 1 second
                frappe.logger().warning(
                    f"Slow query detected: {execution_time}s"
                )
    ```

## API Development

91. **Q: How do you implement REST APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

92. **Q: How do you implement API authentication?**
    ```python
    def authenticate_api():
        def validate_api_key():
            api_key = frappe.get_request_header(
                "Authorization"
            )
            
            if not api_key:
                frappe.throw(
                    "API key required",
                    frappe.AuthenticationError
                )
            
            if not frappe.db.exists(
                "API Key",
                {
                    "api_key": api_key,
                    "enabled": 1
                }
            ):
                frappe.throw(
                    "Invalid API key",
                    frappe.AuthenticationError
                )
    ```

93. **Q: How do you implement API rate limiting?**
    ```python
    def implement_rate_limiting():
        def check_rate_limit():
            user = frappe.session.user
            cache_key = f"rate_limit:{user}"
            
            # Get current count
            count = frappe.cache().get_value(cache_key) or 0
            
            if count >=
# 100 Advanced Frappe Framework Interview Questions and Answers

## Frappe Basics and Architecture

1. **Q: What is Frappe Framework and what are its key features?**
   - A: Frappe is a full-stack web application framework built on Python and JavaScript. Key features include:
   - Meta-data driven architecture
   - Built-in ORM
   - Role-based permissions
   - REST API
   - Real-time updates
   - Extensible architecture
   - Document versioning
   - Background jobs
   - Built-in web forms
   - Multi-tenancy support

2. **Q: How does Frappe's architecture work?**
   ```python
   # Example of Frappe's MVC-like architecture
   # Model (DocType)
   class Customer(Document):
       def validate(self):
           self.validate_credit_limit()
   
   # Controller (Python API)
   @frappe.whitelist()
   def get_customer_details(customer_id):
       return frappe.get_doc("Customer", customer_id)
   
   # View (Form/List)
   # customer.js
   frappe.ui.form.on('Customer', {
       refresh: function(frm) {
           // Form rendering logic
       }
   })
   ```

3. **Q: Explain Frappe's database structure.**
   - A: Frappe uses MariaDB/MySQL with key components:
   ```sql
   -- Main DocType table
   CREATE TABLE `tabCustomer` (
       `name` varchar(140) PRIMARY KEY,
       `creation` datetime,
       `modified` datetime,
       `modified_by` varchar(140),
       `owner` varchar(140),
       `docstatus` int(1) DEFAULT 0,
       -- Custom fields follow
   );
   
   -- Single DocType
   CREATE TABLE `tabSingles` (
       `doctype` varchar(140),
       `field` varchar(140),
       `value` text
   );
   ```

4. **Q: How do you implement custom apps in Frappe?**
   ```bash
   # Creating new app
   bench new-app my_custom_app
   
   # App structure
   my_custom_app/
   ├── MANIFEST.in
   ├── README.md
   ├── my_custom_app/
   │   ├── __init__.py
   │   ├── hooks.py
   │   ├── modules.txt
   │   ├── patches.txt
   │   └── templates/
   └── setup.py
   ```

5. **Q: How do you handle hooks in Frappe?**
   ```python
   # hooks.py
   app_name = "my_custom_app"
   app_title = "My Custom App"
   
   doc_events = {
       "Sales Order": {
           "on_submit": "my_custom_app.events.so_on_submit",
           "on_cancel": "my_custom_app.events.so_on_cancel"
       }
   }
   
   scheduler_events = {
       "daily": [
           "my_custom_app.tasks.daily_cleanup"
       ]
   }
   ```

## DocTypes and Forms

6. **Q: How do you create custom DocTypes?**
   ```json
   {
       "name": "Custom DocType",
       "doctype": "DocType",
       "module": "My Custom App",
       "custom": 1,
       "fields": [
           {
               "fieldname": "title",
               "fieldtype": "Data",
               "label": "Title",
               "reqd": 1
           }
       ],
       "permissions": [
           {
               "role": "System Manager",
               "read": 1,
               "write": 1,
               "create": 1,
               "delete": 1
           }
       ]
   }
   ```

7. **Q: How do you implement custom form validations?**
   ```python
   # Python controller
   class CustomDoc(Document):
       def validate(self):
           if self.end_date < self.start_date:
               frappe.throw("End date cannot be before start date")
   
   # Client-side validation
   frappe.ui.form.on('Custom Doc', {
       validate: function(frm) {
           if (frm.doc.amount < 0) {
               frappe.throw(__("Amount cannot be negative"));
           }
       }
   });
   ```

8. **Q: How do you implement custom fields?**
   ```python
   # Adding custom field programmatically
   def create_custom_field():
       custom_field = frappe.get_doc({
           "doctype": "Custom Field",
           "dt": "Customer",
           "fieldname": "custom_field",
           "fieldtype": "Data",
           "label": "Custom Field",
           "insert_after": "customer_name"
       }).insert()
   ```

9. **Q: How do you implement child tables?**
   ```json
   {
       "name": "Parent DocType",
       "doctype": "DocType",
       "fields": [
           {
               "fieldname": "items",
               "fieldtype": "Table",
               "label": "Items",
               "options": "Child DocType"
           }
       ]
   }
   ```

10. **Q: How do you implement form scripts?**
    ```javascript
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            frm.add_custom_button(__('Custom Action'), function() {
                // Custom action logic
            });
        },
        
        validate: function(frm) {
            // Validation logic
        },
        
        before_save: function(frm) {
            // Pre-save logic
        }
    });
    ```

## Server Scripts and API

11. **Q: How do you implement server-side scripts?**
    ```python
    # Server script for custom validation
    def validate(doc, method):
        if doc.status == "Completed" and not doc.completion_date:
            frappe.throw("Completion date is mandatory")
    
    # Custom API method
    @frappe.whitelist()
    def custom_method(param1, param2):
        # Method logic
        return {"status": "success"}
    ```

12. **Q: How do you implement custom REST APIs?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def get_data():
        return frappe.get_list("DocType",
            fields=["name", "creation"],
            filters={"published": 1}
        )
    
    # Custom API endpoint
    @frappe.whitelist()
    def update_status(docname, status):
        doc = frappe.get_doc("DocType", docname)
        doc.status = status
        doc.save()
        return doc
    ```

13. **Q: How do you implement background jobs?**
    ```python
    # Queuing a background job
    def enqueue_job():
        frappe.enqueue(
            'my_custom_app.tasks.long_running_task',
            queue='long',
            timeout=300,
            doc_name=self.name
        )
    
    # Background job implementation
    def long_running_task(doc_name):
        # Long running process
        frappe.publish_realtime(
            'task_progress',
            {'progress': 50, 'doc_name': doc_name}
        )
    ```

14. **Q: How do you implement custom commands?**
    ```python
    # Custom bench command
    import click
    from frappe.commands import pass_context
    
    @click.command('custom-command')
    @pass_context
    def custom_command(context):
        """Custom command description"""
        frappe.init(site=context.sites[0])
        frappe.connect()
        # Command logic
        frappe.destroy()
    
    commands = [custom_command]
    ```

15. **Q: How do you implement document events?**
    ```python
    # Document event handlers
    def on_update(doc, method):
        frappe.msgprint(f"Document {doc.name} updated")
    
    def on_trash(doc, method):
        # Cleanup related records
        frappe.delete_doc("Related DocType", 
                         filters={"parent": doc.name})
    ```

## Client Scripts and Events

16. **Q: How do you implement client-side scripting?**
    ```javascript
    // Client script
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            // Add custom button
            frm.add_custom_button(__('Process'), function() {
                process_document(frm);
            });
        },
        
        custom_field: function(frm) {
            // Field change handler
            calculate_totals(frm);
        }
    });
    
    function process_document(frm) {
        frappe.call({
            method: 'my_custom_app.api.process_doc',
            args: {
                docname: frm.doc.name
            },
            callback: function(r) {
                frappe.msgprint(r.message);
            }
        });
    }
    ```

17. **Q: How do you implement custom dialogs?**
    ```javascript
    function show_custom_dialog() {
        let d = new frappe.ui.Dialog({
            title: 'Enter details',
            fields: [
                {
                    label: 'First Name',
                    fieldname: 'first_name',
                    fieldtype: 'Data'
                },
                {
                    label: 'Last Name',
                    fieldname: 'last_name',
                    fieldtype: 'Data'
                }
            ],
            primary_action_label: 'Submit',
            primary_action(values) {
                // Handle submission
                d.hide();
            }
        });
        
        d.show();
    }
    ```

18. **Q: How do you implement custom list views?**
    ```javascript
    frappe.listview_settings['DocType'] = {
        add_fields: ['status', 'priority'],
        get_indicator: function(doc) {
            if (doc.status === "Completed") {
                return [__("Completed"), "green", "status,=,Completed"];
            }
        },
        onload: function(listview) {
            // Custom list view logic
        }
    };
    ```

19. **Q: How do you implement custom workspaces?**
    ```python
    # workspace.py
    def get_data():
        return {
            'cards': [
                {
                    'label': 'Sales',
                    'items': [
                        {
                            'type': 'doctype',
                            'name': 'Sales Order',
                            'label': 'Sales Orders'
                        }
                    ]
                }
            ],
            'charts': [
                {
                    'name': 'Sales Analytics',
                    'chart_name': 'Sales Trends'
                }
            ]
        }
    ```

20. **Q: How do you implement custom reports?**
    ```python
    from frappe import _
    
    def execute(filters=None):
        columns = get_columns()
        data = get_data(filters)
        return columns, data
    
    def get_columns():
        return [
            {
                "fieldname": "name",
                "label": _("Name"),
                "fieldtype": "Link",
                "options": "DocType",
                "width": 140
            }
        ]
    ```

## Reports and Queries

21. **Q: How do you implement custom query reports?**
    ```python
    # query_report.py
    def get_data(filters):
        return frappe.db.sql("""
            SELECT 
                name, customer_name, grand_total
            FROM 
                `tabSales Invoice`
            WHERE 
                posting_date BETWEEN %(from_date)s AND %(to_date)s
        """, filters, as_dict=1)
    ```

22. **Q: How do you implement script reports?**
    ```python
    # script_report.py
    class CustomReport(object):
        def __init__(self, filters=None):
            self.filters = filters
        
        def run(self):
            columns = self.get_columns()
            data = self.get_data()
            return columns, data
        
        def get_columns(self):
            return [
                {"label": "ID", "fieldname": "id", "width": 100},
                {"label": "Name", "fieldname": "name", "width": 200}
            ]
    ```

23. **Q: How do you implement custom filters?**
    ```javascript
    // Custom filter
    frappe.query_reports["Custom Report"] = {
        "filters": [
            {
                "fieldname": "from_date",
                "label": __("From Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.add_months(
                    frappe.datetime.get_today(), -1
                ),
                "reqd": 1
            },
            {
                "fieldname": "to_date",
                "label": __("To Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.get_today(),
                "reqd": 1
            }
        ]
    };
    ```

24. **Q: How do you implement custom queries?**
    ```python
    # Custom database queries
    def get_custom_data():
        return frappe.db.sql("""
            SELECT 
                t1.name, 
                t1.customer_name,
                t2.item_code,
                t2.qty
            FROM 
                `tabSales Order` t1
                LEFT JOIN `tabSales Order Item` t2 
                ON t1.name = t2.parent
            WHERE 
                t1.docstatus = 1
        """, as_dict=True)
    ```

25. **Q: How do you implement report charts?**
    ```python
    def get_chart_data(data):
        labels = []
        values = []
        
        for d in data:
            labels.append(d.get("month"))
            values.append(d.get("amount"))
        
        return {
            "data": {
                "labels": labels,
                "datasets": [
                    {
                        "name": "Monthly Sales",
                        "values": values
                    }
                ]
            },
            "type": "line"
        }
    ```

## Customization and Extensions

26. **Q: How do you implement custom apps?**
    ```python
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    app_include_css = [
        "/assets/custom_app/css/custom.css"
    ]
    ```

27. **Q: How do you implement custom fields programmatically?**
    ```python
    def create_custom_fields():
        custom_fields = {
            "Customer": [
                {
                    "fieldname": "custom_field",
                    "label": "Custom Field",
                    "fieldtype": "Data",
                    "insert_after": "customer_name"
                }
            ]
        }
        
        for doctype, fields in custom_fields.items():
            for field in fields:
                if not frappe.db.exists(
                    "Custom Field", 
                    {"dt": doctype, "fieldname": field["fieldname"]}
                ):
                    frappe.get_doc({
                        "doctype": "Custom Field",
                        "dt": doctype,
                        **field
                    }).insert()
    ```

28. **Q: How do you implement custom roles and permissions?**
    ```python
    def create_custom_role():
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Role"
            role.desk_access = 1
            role.insert()
    
    def set_custom_permissions():
        # Custom DocPerm
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.read = 1
        docperm.write = 1
        docperm.create = 1
        docperm.submit = 0
        docperm.cancel = 0
        docperm.delete = 0
        docperm.insert()
    ```

29. **Q: How do you implement custom print formats?**
    ```html
    <!-- custom_format.html -->
    <div class="print-format">
        <div class="row">
            <div class="col-xs-6">
                <div class="row">
                    <div class="col-xs-5">
                        <img src="{{ letter_head }}">
                    </div>
                </div>
            </div>
        </div>
        
        <div class="row">
            <div class="col-xs-12">
                <div class="row">
                    <div class="col-xs-6">
                        <strong>{{ doc.customer_name }}</strong><br>
                        {{ doc.address_display }}
                    </div>
                </div>
            </div>
        </div>
    </div>
    ```

30. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.new_doc("Workflow")
        workflow.workflow_name = "Custom Workflow"
        workflow.document_type = "Custom DocType"
        workflow.workflow_state_field = "workflow_state"
        
        # States
        workflow.states = [
            {
                "state": "Draft",
                "doc_status": 0,
                "allow_edit": "Custom Role"
            },
            {
                "state": "Submitted",
                "doc_status": 1,
                "allow_edit": "System Manager"
            }
        ]
        
        # Transitions
        workflow.transitions = [
            {
                "state": "Draft",
                "action": "Submit",
                "next_state": "Submitted",
                "allowed": "Custom Role"
            }
        ]
        
        workflow.insert()
    ```

## Workflow and Automation

31. **Q: How do you implement custom workflows with multiple states?**
    ```python
    def setup_workflow():
        workflow = {
            "name": "Custom Process",
            "document_type": "Custom DocType",
            "states": [
                {
                    "state": "Draft",
                    "style": "Primary",
                    "doc_status": 0
                },
                {
                    "state": "Pending Approval",
                    "style": "Warning",
                    "doc_status": 0
                },
                {
                    "state": "Approved",
                    "style": "Success",
                    "doc_status": 1
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Custom User"
                },
                {
                    "state": "Pending Approval",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "Custom Approver"
                }
            ]
        }
        
        if not frappe.db.exists("Workflow", workflow["name"]):
            doc = frappe.get_doc({"doctype": "Workflow", **workflow})
            doc.insert()
    ```

32. **Q: How do you implement scheduled tasks?**
    ```python
    # hooks.py
    scheduler_events = {
        "daily": [
            "custom_app.tasks.daily_cleanup"
        ],
        "hourly": [
            "custom_app.tasks.send_reminders"
        ],
        "weekly": [
            "custom_app.tasks.weekly_report"
        ]
    }
    
    # tasks.py
    def daily_cleanup():
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    
    def send_reminders():
        # Send reminder emails
        pass
    ```

33. **Q: How do you implement document event handlers?**
    ```python
    # hooks.py
    doc_events = {
        "Sales Order": {
            "on_submit": "custom_app.events.so_on_submit",
            "on_cancel": "custom_app.events.so_on_cancel",
            "after_insert": "custom_app.events.so_after_insert"
        }
    }
    
    # events.py
    def so_on_submit(doc, method):
        # Create related documents
        create_project(doc)
        notify_departments(doc)
    
    def notify_departments(doc):
        frappe.publish_realtime(
            'sales_order_submitted',
            {"sales_order": doc.name}
        )
    ```

34. **Q: How do you implement custom notifications?**
    ```python
    def setup_notification():
        notification = frappe.new_doc("Notification")
        notification.name = "Custom Alert"
        notification.subject = "New {{ doc.doctype }} {{ doc.name }}"
        notification.document_type = "Custom DocType"
        notification.event = "Submit"
        notification.channel = "Email"
        notification.message = """
            Dear {{ doc.customer_name }},
            Your {{ doc.doctype }} {{ doc.name }} has been submitted.
        """
        notification.insert()
    ```

35. **Q: How do you implement email alerts?**
    ```python
    def send_custom_email(doc):
        frappe.sendmail(
            recipients=[doc.email],
            subject=f"Document {doc.name} Status Update",
            template="custom_email",
            args={
                "name": doc.name,
                "status": doc.status,
                "comments": doc.comments
            },
            header=["Status Update", "green"]
        )
    
    # custom_email.html
    <h3>Status Update</h3>
    <p>Document {{ name }} status has been updated to {{ status }}</p>
    {% if comments %}
    <p>Comments: {{ comments }}</p>
    {% endif %}
    ```

## Security and Permissions

36. **Q: How do you implement role-based access control?**
    ```python
    def setup_roles_and_permissions():
        # Create custom role
        if not frappe.db.exists("Role", "Custom Manager"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Manager"
            role.desk_access = 1
            role.insert()
        
        # Set permissions
        custom_manager_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        add_permission(
            "Custom DocType",
            "Custom Manager",
            custom_manager_perms
        )
    ```

37. **Q: How do you implement custom authentication?**
    ```python
    from frappe.auth import LoginManager
    
    class CustomLoginManager(LoginManager):
        def validate_user(self):
            # Custom validation logic
            super().validate_user()
            
            if not self.user_in_approved_list():
                frappe.throw("User not approved")
        
        def user_in_approved_list(self):
            return frappe.db.exists(
                "Approved Users",
                {"user": self.user}
            )
    ```

38. **Q: How do you implement field-level permissions?**
    ```python
    def setup_field_permissions():
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.permlevel = 1
        docperm.read = 1
        docperm.write = 0
        
        # Field permission in DocType
        custom_field = {
            "fieldname": "sensitive_data",
            "permlevel": 1
        }
        
        update_field_permission(
            "Custom DocType",
            custom_field
        )
    ```

39. **Q: How do you implement API authentication?**
    ```python
    @frappe.whitelist()
    def custom_api():
        # Check if user has permission
        if not frappe.has_permission("Custom DocType"):
            frappe.throw("Not permitted")
        
        # API token validation
        api_token = frappe.get_request_header("Authorization")
        if not validate_api_token(api_token):
            frappe.throw("Invalid API token")
        
        return "API response"
    
    def validate_api_token(token):
        return frappe.db.exists(
            "API Token",
            {"token": token, "enabled": 1}
        )
    ```

40. **Q: How do you implement user permissions?**
    ```python
    def setup_user_permissions():
        # Add user permission
        user_permission = frappe.new_doc("User Permission")
        user_permission.user = "user@example.com"
        user_permission.allow = "Custom DocType"
        user_permission.for_value = "CUST-001"
        user_permission.apply_to_all_doctypes = 1
        user_permission.insert()
        
        # Check user permission
        def validate_user_permission(doc, method):
            if not frappe.has_permission(
                doc.doctype,
                "write",
                doc
            ):
                frappe.throw("Not permitted")
    ```

## Deployment and Performance

41. **Q: How do you implement custom deployment configurations?**
    ```python
    # config.py
    from frappe.utils import cstr, cint
    
    def get_site_config():
        return {
            'maintenance_mode': 0,
            'pause_scheduler': 0,
            'developer_mode': 0,
            'disable_website_cache': 0,
            'rate_limit': {
                'limit': 100,
                'window': 3600
            }
        }
    ```

42. **Q: How do you implement caching strategies?**
    ```python
    from frappe.utils.caching import redis_cache
    
    @redis_cache
    def get_cached_data(key):
        # Expensive operation
        result = frappe.db.sql("""
            SELECT * FROM `tabCustom DocType`
            WHERE status = 'Active'
        """, as_dict=True)
        return result
    
    def clear_cache():
        frappe.cache().delete_key('cached_data')
    ```

43. **Q: How do you implement database optimization?**
    ```python
    def optimize_database():
        # Add indexes
        frappe.db.sql("""
            CREATE INDEX IF NOT EXISTS idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize tables
        frappe.db.sql("""
            OPTIMIZE TABLE `tabCustom DocType`
        """)
        
        # Clear unused data
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

44. **Q: How do you implement load balancing?**
    ```python
    # nginx configuration
    def get_nginx_config():
        return """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
        """
    ```

45. **Q: How do you implement monitoring and logging?**
    ```python
    import logging
    
    def setup_logging():
        # Custom logger
        logger = logging.getLogger("custom_app")
        logger.setLevel(logging.DEBUG)
        
        # File handler
        fh = logging.FileHandler("custom_app.log")
        fh.setLevel(logging.DEBUG)
        
        # Formatter
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        fh.setFormatter(formatter)
        logger.addHandler(fh)
        
        return logger
    
    # Usage
    logger = setup_logging()
    logger.info("Custom operation completed")
    ```

## Integration and Development

46. **Q: How do you implement external API integration?**
    ```python
    import requests
    
    class ExternalAPIIntegration:
        def __init__(self):
            self.api_key = frappe.conf.get("external_api_key")
            self.base_url = "https://api.external.com/v1"
        
        def make_request(self, endpoint, method="GET", data=None):
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            response = requests.request(
                method,
                f"{self.base_url}/{endpoint}",
                headers=headers,
                json=data
            )
            
            response.raise_for_status()
            return response.json()
    ```

47. **Q: How do you implement webhooks?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def webhook_handler():
        if not validate_webhook_signature():
            frappe.throw("Invalid webhook signature")
        
        data = frappe.request.get_json()
        
        # Process webhook data
        process_webhook_data(data)
        
        return "Success"
    
    def validate_webhook_signature():
        signature = frappe.get_request_header("X-Webhook-Signature")
        # Validate signature
        return True
    ```

48. **Q: How do you implement custom REST endpoints?**
    ```python
    @frappe.whitelist()
    def custom_endpoint():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

49. **Q: How do you implement file handling?**
    ```python
    def handle_file_upload():
        file_doc = frappe.get_doc({
            "doctype": "File",
            "attached_to_doctype": "Custom DocType",
            "attached_to_name": "DOC-001",
            "attached_to_field": "attachment",
            "file_url": "",
            "file_name": "document.pdf",
            "is_private": 1,
            "content": frappe.request.files['file']
        }).insert()
        
        return file_doc
    ```

50. **Q: How do you implement custom apps and modules?**
    ```python
    # modules.txt
    Custom App
    Core
    Integration
    
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    ```

## ERPNext Integration

51. **Q: How do you extend ERPNext DocTypes?**
    ```python
    # custom_sales_order.py
    from erpnext.selling.doctype.sales_order.sales_order import SalesOrder
    
    class CustomSalesOrder(SalesOrder):
        def validate(self):
            super().validate()
            self.validate_custom_fields()
        
        def validate_custom_fields(self):
            if self.custom_field < 0:
                frappe.throw("Custom field cannot be negative")
    ```

52. **Q: How do you implement custom pricing rules?**
    ```python
    def get_custom_price(item_code, customer):
        pricing_rule = frappe.get_doc({
            "doctype": "Pricing Rule",
            "title": f"Custom Price for {customer}",
            "apply_on": "Item Code",
            "items": [{
                "item_code": item_code
            }],
            "selling": 1,
            "rate_or_discount": "Rate",
            "rate": 100
        })
        return pricing_rule
    ```

53. **Q: How do you implement custom workflows in ERPNext?**
    ```python
    def create_custom_workflow():
        workflow = {
            "name": "Custom SO Workflow",
            "document_type": "Sales Order",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Sales User"
                },
                {
                    "state": "Pending Approval",
                    "doc_status": 0,
                    "allow_edit": "Sales Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Sales User"
                }
            ]
        }
        return workflow
    ```

54. **Q: How do you implement custom reports in ERPNext?**
    ```python
    def get_custom_report():
        return {
            "columns": [
                {
                    "fieldname": "customer",
                    "label": "Customer",
                    "fieldtype": "Link",
                    "options": "Customer",
                    "width": 180
                },
                {
                    "fieldname": "total_sales",
                    "label": "Total Sales",
                    "fieldtype": "Currency",
                    "width": 140
                }
            ],
            "filters": [
                {
                    "fieldname": "from_date",
                    "label": "From Date",
                    "fieldtype": "Date",
                    "default": frappe.utils.add_months(
                        frappe.utils.nowdate(), -1
                    )
                }
            ]
        }
    ```

55. **Q: How do you implement custom item variants?**
    ```python
    def create_item_variant():
        template = frappe.get_doc("Item", "ITEM-TEMP-001")
        variant = frappe.get_doc({
            "doctype": "Item",
            "variant_of": template.name,
            "item_code": "ITEM-VAR-001",
            "item_name": "Custom Variant",
            "item_group": template.item_group,
            "stock_uom": template.stock_uom,
            "attributes": [
                {
                    "attribute": "Size",
                    "attribute_value": "Large"
                }
            ]
        }).insert()
        return variant
    ```

## Advanced Development

56. **Q: How do you implement custom web pages?**
    ```python
    # hooks.py
    website_route_rules = [
        {"from_route": "/custom", "to_route": "custom/index"}
    ]
    
    # custom/index.html
    {% extends "templates/web.html" %}
    
    {% block page_content %}
    <div class="custom-page">
        <h1>{{ title }}</h1>
        {{ content }}
    </div>
    {% endblock %}
    
    # custom/index.py
    def get_context(context):
        context.title = "Custom Page"
        context.content = frappe.get_doc(
            "Custom Content", 
            "CONTENT-001"
        ).content
    ```

57. **Q: How do you implement custom portals?**
    ```python
    # hooks.py
    portal_menu_items = [
        {
            "title": "Custom Portal",
            "route": "/custom-portal",
            "reference_doctype": "Custom DocType",
            "role": "Customer"
        }
    ]
    
    # custom_portal.js
    frappe.pages['custom-portal'].on_page_load = function(wrapper) {
        var page = frappe.ui.make_app_page({
            parent: wrapper,
            title: 'Custom Portal',
            single_column: true
        });
        
        page.add_inner_button(__('New'), function() {
            frappe.new_doc('Custom DocType');
        });
    }
    ```

58. **Q: How do you implement custom dashboards?**
    ```python
    def get_dashboard_data(data):
        return {
            'fieldname': 'custom_doctype',
            'transactions': [
                {
                    'label': _('Related Documents'),
                    'items': ['Sales Order', 'Purchase Order']
                }
            ],
            'charts': [
                {
                    'name': 'Custom Chart',
                    'chart_name': 'Monthly Trends',
                    'chart_type': 'Line'
                }
            ]
        }
    ```

59. **Q: How do you implement custom scripts?**
    ```python
    # Server Script
    doc = frappe.get_doc({
        "doctype": "Server Script",
        "name": "Custom Server Script",
        "script_type": "DocType Event",
        "doc_type": "Custom DocType",
        "event": "Before Save",
        "script": """
    if doc.status == "Completed" and not doc.completion_date:
        doc.completion_date = frappe.utils.nowdate()
        """
    })
    
    # Client Script
    doc = frappe.get_doc({
        "doctype": "Client Script",
        "name": "Custom Client Script",
        "dt": "Custom DocType",
        "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
        """
    })
    ```

60. **Q: How do you implement custom APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    
    def process_request(params):
        # Custom processing logic
        return {
            "processed": True,
            "params": params
        }
    ```

## Advanced Features

61. **Q: How do you implement custom email templates?**
    ```python
    def create_email_template():
        template = frappe.get_doc({
            "doctype": "Email Template",
            "name": "Custom Template",
            "subject": "{{ doc.name }} - Status Update",
            "response": """
                Dear {{ doc.customer_name }},
                
                Your {{ doc.doctype }} ({{ doc.name }}) 
                has been updated to {{ doc.status }}.
                
                Regards,
                {{ frappe.defaults.get_global_default('company') }}
            """
        }).insert()
        return template
    ```

62. **Q: How do you implement custom print formats?**
    ```python
    def create_print_format():
        format_doc = frappe.get_doc({
            "doctype": "Print Format",
            "name": "Custom Format",
            "doc_type": "Custom DocType",
            "standard": "No",
            "print_format_type": "Jinja",
            "html": """
                <div class="print-heading">
                    <h2>{{ doc.name }}</h2>
                </div>
                
                <div class="row">
                    <div class="col-xs-6">
                        <div class="row">
                            <div class="col-xs-5">
                                <label>Customer</label>
                            </div>
                            <div class="col-xs-7">
                                {{ doc.customer_name }}
                            </div>
                        </div>
                    </div>
                </div>
            """
        }).insert()
        return format_doc
    ```

63. **Q: How do you implement custom web forms?**
    ```python
    def create_web_form():
        web_form = frappe.get_doc({
            "doctype": "Web Form",
            "title": "Custom Form",
            "route": "custom-form",
            "doc_type": "Custom DocType",
            "web_form_fields": [
                {
                    "fieldname": "customer_name",
                    "fieldtype": "Data",
                    "label": "Customer Name",
                    "reqd": 1
                },
                {
                    "fieldname": "email",
                    "fieldtype": "Data",
                    "label": "Email",
                    "reqd": 1,
                    "options": "Email"
                }
            ]
        }).insert()
        return web_form
    ```

64. **Q: How do you implement custom roles and permissions?**
    ```python
    def setup_custom_role():
        # Create role
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.get_doc({
                "doctype": "Role",
                "role_name": "Custom Role",
                "desk_access": 1
            }).insert()
        
        # Set permissions
        custom_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        if not frappe.db.exists(
            "Custom DocPerm",
            {
                "parent": "Custom DocType",
                "role": "Custom Role"
            }
        ):
            perm = frappe.get_doc({
                "doctype": "Custom DocPerm",
                "parent": "Custom DocType",
                "parenttype": "DocType",
                "parentfield": "permissions",
                "role": "Custom Role",
                **custom_perms
            }).insert()
    ```

65. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.get_doc({
            "doctype": "Workflow",
            "workflow_name": "Custom Process",
            "document_type": "Custom DocType",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Custom Role"
                },
                {
                    "state": "Pending",
                    "doc_status": 0,
                    "allow_edit": "System Manager"
                },
                {
                    "state": "Approved",
                    "doc_status": 1,
                    "allow_edit": "System Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit",
                    "next_state": "Pending",
                    "allowed": "Custom Role"
                },
                {
                    "state": "Pending",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "System Manager"
                }
            ]
        }).insert()
        return workflow
    ```

## Testing and Debugging

66. **Q: How do you implement unit tests?**
    ```python
    import unittest
    
    class TestCustomDoc(unittest.TestCase):
        def setUp(self):
            # Create test data
            self.doc = frappe.get_doc({
                "doctype": "Custom DocType",
                "title": "Test Doc"
            }).insert()
        
        def tearDown(self):
            # Cleanup test data
            frappe.delete_doc(
                "Custom DocType",
                self.doc.name
            )
        
        def test_validation(self):
            self.doc.status = "Completed"
            self.assertRaises(
                frappe.ValidationError,
                self.doc.save
            )
    ```

67. **Q: How do you implement integration tests?**
    ```python
    class TestIntegration(unittest.TestCase):
        def test_api_integration(self):
            # Test API endpoint
            response = frappe.call(
                'custom_app.api.custom_endpoint',
                param1="test",
                param2="value"
            )
            
            self.assertEqual(
                response.get("status"),
                "success"
            )
        
        def test_workflow(self):
            # Test workflow transitions
            doc = create_test_doc()
            doc.submit()
            
            self.assertEqual(
                doc.workflow_state,
                "Pending Approval"
            )
    ```

68. **Q: How do you implement custom error handling?**
    ```python
    class CustomError(Exception):
        def __init__(self, message, error_code=None):
            super().__init__(message)
            self.error_code = error_code
    
    def custom_error_handler():
        try:
            # Risky operation
            result = process_data()
            
            if not result:
                raise CustomError(
                    "Processing failed",
                    "PROC001"
                )
            
            return result
            
        except CustomError as e:
            frappe.log_error(
                title="Custom Error",
                message=str(e)
            )
            frappe.throw(
                str(e),
                exc=e.__class__
            )
    ```

69. **Q: How do you implement debugging tools?**
    ```python
    def debug_document(doc):
        # Log document state
        frappe.logger().debug({
            "doctype": doc.doctype,
            "name": doc.name,
            "status": doc.status,
            "modified": doc.modified
        })
        
        # Track changes
        frappe.logger().debug({
            "changed_fields": doc.get_changed(),
            "user": frappe.session.user,
            "timestamp": frappe.utils.now()
        })
    ```

70. **Q: How do you implement performance profiling?**
    ```python
    import cProfile
    import pstats
    
    def profile_function():
        profiler = cProfile.Profile()
        profiler.enable()
        
        # Function to profile
        result = expensive_operation()
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats()
        
        return result
    ```

## Advanced Customization

71. **Q: How do you implement custom field types?**
    ```python
    class CustomField(frappe.model.document.Document):
        def validate(self):
            if self.fieldtype == "Custom":
                self.validate_custom_field()
        
        def validate_custom_field(self):
            # Custom validation logic
            if not self.options:
                frappe.throw(
                    "Options required for Custom field type"
                )
    ```

72. **Q: How do you implement custom controllers?**
    ```python
    class CustomController(frappe.model.document.Document):
        def validate(self):
            self.validate_custom_logic()
        
        def on_submit(self):
            self.create_related_docs()
        
        def validate_custom_logic(self):
            if self.end_date <= self.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
        
        def create_related_docs(self):
            # Create related documents
            self.create_project()
            self.create_tasks()
    ```

73. **Q: How do you implement custom doctypes?**
    ```python
    def create_custom_doctype():
        doctype = frappe.get_doc({
            "doctype": "DocType",
            "name": "Custom DocType",
            "module": "Custom App",
            "custom": 1,
            "fields": [
                {
                    "fieldname": "title",
                    "fieldtype": "Data",
                    "label": "Title",
                    "reqd": 1
                },
                {
                    "fieldname": "status",
                    "fieldtype": "Select",
                    "label": "Status",
                    "options": "Draft\nPending\nCompleted"
                }
            ],
            "permissions": [
                {
                    "role": "System Manager",
                    "read": 1,
                    "write": 1,
                    "create": 1,
                    "delete": 1
                }
            ]
        }).insert()
        return doctype
    ```

74. **Q: How do you implement custom scripts?**
    ```python
    def create_custom_scripts():
        # Server script
        server_script = frappe.get_doc({
            "doctype": "Server Script",
            "name": "Custom Server Script",
            "script_type": "DocType Event",
            "doc_type": "Custom DocType",
            "event": "Before Save",
            "script": """
    if doc.status == "Completed":
        doc.completion_date = frappe.utils.now()
            """
        }).insert()
        
        # Client script
        client_script = frappe.get_doc({
            "doctype": "Client Script",
            "name": "Custom Client Script",
            "dt": "Custom DocType",
            "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
            """
        }).insert()
    ```

75. **Q: How do you implement custom reports?**
    ```python
    def create_custom_report():
        report = frappe.get_doc({
            "doctype": "Report",
            "name": "Custom Report",
            "ref_doctype": "Custom DocType",
            "report_type": "Script Report",
            "is_standard": "No",
            "module": "Custom App",
            "query": """
    SELECT
        name,
        creation,
        modified,
        status
    FROM
        `tabCustom DocType`
    WHERE
        status = %(status)s
            """,
            "filters": [
                {
                    "fieldname": "status",
                    "label": "Status",
                    "fieldtype": "Select",
                    "options": "Draft\nPending\nCompleted",
                    "default": "Pending"
                }
            ]
        }).insert()
        return report
    ```

## Data Management

76. **Q: How do you implement data import/export?**
    ```python
    def import_data():
        from frappe.core.doctype.data_import.data_import import (
            import_file
        )
        
        # Import data
        import_file(
            doctype="Custom DocType",
            file_path="path/to/import.csv",
            import_type="Insert",
            submit_after_import=True
        )
    
    def export_data():
        from frappe.desk.query_report import export_query
        
        # Export data
        export_query(
            "Custom Report",
            "CSV",
            filters={"status": "Completed"}
        )
    ```

77. **Q: How do you implement data migrations?**
    ```python
    def migrate_data():
        # Create migration plan
        plan = frappe.get_doc({
            "doctype": "Data Migration Plan",
            "name": "Custom Migration",
            "module": "Custom App",
            "mappings": [
                {
                    "mapping": "Customer to Contact",
                    "remote_objectname": "Contact",
                    "page_length": 10
                }
            ]
        })
        
        # Run migration
        run = frappe.get_doc({
            "doctype": "Data Migration Run",
            "data_migration_plan": plan.name
        }).insert()
        
        run.run()
    ```

78. **Q: How do you implement data archival?**
    ```python
    def archive_data():
        # Archive old records
        frappe.db.sql("""
            INSERT INTO `tabArchived DocType`
            SELECT *
            FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
        
        # Delete archived records
        frappe.db.sql("""
            DELETE FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
    ```

79. **Q: How do you implement data validation?**
    ```python
    def validate_data():
        def validate_custom_doc(doc, method):
            # Validate required fields
            if not doc.title:
                frappe.throw("Title is required")
            
            # Validate business logic
            if doc.end_date <= doc.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
            
            # Validate uniqueness
            if frappe.db.exists(
                "Custom DocType",
                {
                    "title": doc.title,
                    "name": ("!=", doc.name)
                }
            ):
                frappe.throw("Title must be unique")
    ```

80. **Q: How do you implement data cleanup?**
    ```python
    def cleanup_data():
        # Delete temporary files
        frappe.db.sql("""
            DELETE FROM `tabFile`
            WHERE attached_to_doctype = 'Custom DocType'
            AND creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
        
        # Clear error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 7 DAY)
        """)
        
        # Remove orphaned records
        frappe.db.sql("""
            DELETE FROM `tabCustom Child Table`
            WHERE parent NOT IN (
                SELECT name
                FROM `tabCustom DocType`
            )
        """)
    ```

## System Administration

81. **Q: How do you implement system monitoring?**
    ```python
    def monitor_system():
        # Monitor database
        db_stats = frappe.db.sql("""
            SHOW STATUS
            WHERE Variable_name IN (
                'Threads_connected',
                'Questions',
                'Slow_queries'
            )
        """, as_dict=1)
        
        # Monitor background jobs
        jobs = frappe.db.sql("""
            SELECT status, COUNT(*) as count
            FROM `tabQueue Job`
            GROUP BY status
        """, as_dict=1)
        
        # Log statistics
        frappe.logger().info({
            "db_stats": db_stats,
            "background_jobs": jobs
        })
    ```

82. **Q: How do you implement backup and recovery?**
    ```python
    def manage_backups():
        from frappe.utils.backups import backup
        
        # Create backup
        backup_path = backup(
            with_files=True,
            backup_path_db="/path/to/db/backup",
            backup_path_files="/path/to/files/backup"
        )
        
        # Restore from backup
        def restore_backup():
            # Stop server
            os.system("bench stop")
            
            # Restore database
            os.system(
                f"mysql -u {db_name} < {backup_path}/database.sql"
            )
            
            # Restore files
            os.system(
                f"rsync -a {backup_path}/files/ ./sites/site1.local/public/files/"
            )
            
            # Start server
            os.system("bench start")
    ```

83. **Q: How do you implement system updates?**
    ```python
    def update_system():
        # Update apps
        os.system("bench update")
        
        # Update dependencies
        os.system("bench setup requirements")
        
        # Migrate database
        os.system("bench migrate")
        
        # Clear cache
        frappe.clear_cache()
        
        # Restart server
        os.system("bench restart")
    ```

84. **Q: How do you implement system maintenance?**
    ```python
    def maintain_system():
        # Cleanup temporary files
        cleanup_temporary_files()
        
        # Optimize database
        frappe.db.sql("OPTIMIZE TABLE `tabCustom DocType`")
        
        # Clear cache
        frappe.clear_cache()
        
        # Delete old error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

85. **Q: How do you implement system security?**
    ```python
    def secure_system():
        # Set secure configuration
        frappe.db.set_value(
            "System Settings",
            None,
            {
                "allow_login_using_mobile_number": 0,
                "allow_login_using_user_name": 0,
                "allow_guests_to_upload_files": 0,
                "force_user_to_reset_password": 1
            }
        )
        
        # Enable two-factor authentication
        frappe.db.set_value(
            "System Settings",
            None,
            "enable_two_factor_auth",
            1
        )
    ```

## Performance Optimization

86. **Q: How do you implement caching strategies?**
    ```python
    def implement_caching():
        # Redis cache
        @frappe.whitelist()
        def get_cached_data():
            cache_key = "custom_data"
            data = frappe.cache().get_value(cache_key)
            
            if not data:
                data = get_expensive_data()
                frappe.cache().set_value(
                    cache_key,
                    data,
                    expires_in_sec=3600
                )
            
            return data
    ```

87. **Q: How do you implement query optimization?**
    ```python
    def optimize_queries():
        # Use proper indexes
        frappe.db.sql("""
            CREATE INDEX idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize joins
        results = frappe.db.sql("""
            SELECT t1.name, t2.item_code
            FROM `tabCustom DocType` t1
            INNER JOIN `tabCustom Child Table` t2
            ON t1.name = t2.parent
            WHERE t1.status = 'Active'
        """, as_dict=1)
    ```

88. **Q: How do you implement background jobs?**
    ```python
    def manage_background_jobs():
        # Enqueue long-running job
        frappe.enqueue(
            'custom_app.tasks.process_data',
            queue='long',
            timeout=1500,
            doc_name="DOC-001"
        )
        
        # Monitor job status
        def get_job_status(job_id):
            return frappe.db.get_value(
                'Queue Job',
                {'job_id': job_id},
                'status'
            )
    ```

89. **Q: How do you implement load balancing?**
    ```python
    def configure_load_balancing():
        # Nginx configuration
        nginx_config = """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
            }
        }
        """
    ```

90. **Q: How do you implement performance monitoring?**
    ```python
    def monitor_performance():
        import time
        
        def log_slow_queries():
            start_time = time.time()
            
            # Execute query
            result = frappe.db.sql(
                "SELECT * FROM `tabCustom DocType`"
            )
            
            execution_time = time.time() - start_time
            
            if execution_time > 1.0:  # Log if > 1 second
                frappe.logger().warning(
                    f"Slow query detected: {execution_time}s"
                )
    ```

## API Development

91. **Q: How do you implement REST APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

92. **Q: How do you implement API authentication?**
    ```python
    def authenticate_api():
        def validate_api_key():
            api_key = frappe.get_request_header(
                "Authorization"
            )
            
            if not api_key:
                frappe.throw(
                    "API key required",
                    frappe.AuthenticationError
                )
            
            if not frappe.db.exists(
                "API Key",
                {
                    "api_key": api_key,
                    "enabled": 1
                }
            ):
                frappe.throw(
                    "Invalid API key",
                    frappe.AuthenticationError
                )
    ```

93. **Q: How do you implement API rate limiting?**
    ```python
    def implement_rate_limiting():
        def check_rate_limit():
            user = frappe.session.user
            cache_key = f"rate_limit:{user}"
            
            # Get current count
            count = frappe.cache().get_value(cache_key) or 0
# 100 Advanced Frappe Framework Interview Questions and Answers

## Frappe Basics and Architecture

1. **Q: What is Frappe Framework and what are its key features?**
   - A: Frappe is a full-stack web application framework built on Python and JavaScript. Key features include:
   - Meta-data driven architecture
   - Built-in ORM
   - Role-based permissions
   - REST API
   - Real-time updates
   - Extensible architecture
   - Document versioning
   - Background jobs
   - Built-in web forms
   - Multi-tenancy support

2. **Q: How does Frappe's architecture work?**
   ```python
   # Example of Frappe's MVC-like architecture
   # Model (DocType)
   class Customer(Document):
       def validate(self):
           self.validate_credit_limit()
   
   # Controller (Python API)
   @frappe.whitelist()
   def get_customer_details(customer_id):
       return frappe.get_doc("Customer", customer_id)
   
   # View (Form/List)
   # customer.js
   frappe.ui.form.on('Customer', {
       refresh: function(frm) {
           // Form rendering logic
       }
   })
   ```

3. **Q: Explain Frappe's database structure.**
   - A: Frappe uses MariaDB/MySQL with key components:
   ```sql
   -- Main DocType table
   CREATE TABLE `tabCustomer` (
       `name` varchar(140) PRIMARY KEY,
       `creation` datetime,
       `modified` datetime,
       `modified_by` varchar(140),
       `owner` varchar(140),
       `docstatus` int(1) DEFAULT 0,
       -- Custom fields follow
   );
   
   -- Single DocType
   CREATE TABLE `tabSingles` (
       `doctype` varchar(140),
       `field` varchar(140),
       `value` text
   );
   ```

4. **Q: How do you implement custom apps in Frappe?**
   ```bash
   # Creating new app
   bench new-app my_custom_app
   
   # App structure
   my_custom_app/
   ├── MANIFEST.in
   ├── README.md
   ├── my_custom_app/
   │   ├── __init__.py
   │   ├── hooks.py
   │   ├── modules.txt
   │   ├── patches.txt
   │   └── templates/
   └── setup.py
   ```

5. **Q: How do you handle hooks in Frappe?**
   ```python
   # hooks.py
   app_name = "my_custom_app"
   app_title = "My Custom App"
   
   doc_events = {
       "Sales Order": {
           "on_submit": "my_custom_app.events.so_on_submit",
           "on_cancel": "my_custom_app.events.so_on_cancel"
       }
   }
   
   scheduler_events = {
       "daily": [
           "my_custom_app.tasks.daily_cleanup"
       ]
   }
   ```

## DocTypes and Forms

6. **Q: How do you create custom DocTypes?**
   ```json
   {
       "name": "Custom DocType",
       "doctype": "DocType",
       "module": "My Custom App",
       "custom": 1,
       "fields": [
           {
               "fieldname": "title",
               "fieldtype": "Data",
               "label": "Title",
               "reqd": 1
           }
       ],
       "permissions": [
           {
               "role": "System Manager",
               "read": 1,
               "write": 1,
               "create": 1,
               "delete": 1
           }
       ]
   }
   ```

7. **Q: How do you implement custom form validations?**
   ```python
   # Python controller
   class CustomDoc(Document):
       def validate(self):
           if self.end_date < self.start_date:
               frappe.throw("End date cannot be before start date")
   
   # Client-side validation
   frappe.ui.form.on('Custom Doc', {
       validate: function(frm) {
           if (frm.doc.amount < 0) {
               frappe.throw(__("Amount cannot be negative"));
           }
       }
   });
   ```

8. **Q: How do you implement custom fields?**
   ```python
   # Adding custom field programmatically
   def create_custom_field():
       custom_field = frappe.get_doc({
           "doctype": "Custom Field",
           "dt": "Customer",
           "fieldname": "custom_field",
           "fieldtype": "Data",
           "label": "Custom Field",
           "insert_after": "customer_name"
       }).insert()
   ```

9. **Q: How do you implement child tables?**
   ```json
   {
       "name": "Parent DocType",
       "doctype": "DocType",
       "fields": [
           {
               "fieldname": "items",
               "fieldtype": "Table",
               "label": "Items",
               "options": "Child DocType"
           }
       ]
   }
   ```

10. **Q: How do you implement form scripts?**
    ```javascript
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            frm.add_custom_button(__('Custom Action'), function() {
                // Custom action logic
            });
        },
        
        validate: function(frm) {
            // Validation logic
        },
        
        before_save: function(frm) {
            // Pre-save logic
        }
    });
    ```

## Server Scripts and API

11. **Q: How do you implement server-side scripts?**
    ```python
    # Server script for custom validation
    def validate(doc, method):
        if doc.status == "Completed" and not doc.completion_date:
            frappe.throw("Completion date is mandatory")
    
    # Custom API method
    @frappe.whitelist()
    def custom_method(param1, param2):
        # Method logic
        return {"status": "success"}
    ```

12. **Q: How do you implement custom REST APIs?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def get_data():
        return frappe.get_list("DocType",
            fields=["name", "creation"],
            filters={"published": 1}
        )
    
    # Custom API endpoint
    @frappe.whitelist()
    def update_status(docname, status):
        doc = frappe.get_doc("DocType", docname)
        doc.status = status
        doc.save()
        return doc
    ```

13. **Q: How do you implement background jobs?**
    ```python
    # Queuing a background job
    def enqueue_job():
        frappe.enqueue(
            'my_custom_app.tasks.long_running_task',
            queue='long',
            timeout=300,
            doc_name=self.name
        )
    
    # Background job implementation
    def long_running_task(doc_name):
        # Long running process
        frappe.publish_realtime(
            'task_progress',
            {'progress': 50, 'doc_name': doc_name}
        )
    ```

14. **Q: How do you implement custom commands?**
    ```python
    # Custom bench command
    import click
    from frappe.commands import pass_context
    
    @click.command('custom-command')
    @pass_context
    def custom_command(context):
        """Custom command description"""
        frappe.init(site=context.sites[0])
        frappe.connect()
        # Command logic
        frappe.destroy()
    
    commands = [custom_command]
    ```

15. **Q: How do you implement document events?**
    ```python
    # Document event handlers
    def on_update(doc, method):
        frappe.msgprint(f"Document {doc.name} updated")
    
    def on_trash(doc, method):
        # Cleanup related records
        frappe.delete_doc("Related DocType", 
                         filters={"parent": doc.name})
    ```

## Client Scripts and Events

16. **Q: How do you implement client-side scripting?**
    ```javascript
    // Client script
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            // Add custom button
            frm.add_custom_button(__('Process'), function() {
                process_document(frm);
            });
        },
        
        custom_field: function(frm) {
            // Field change handler
            calculate_totals(frm);
        }
    });
    
    function process_document(frm) {
        frappe.call({
            method: 'my_custom_app.api.process_doc',
            args: {
                docname: frm.doc.name
            },
            callback: function(r) {
                frappe.msgprint(r.message);
            }
        });
    }
    ```

17. **Q: How do you implement custom dialogs?**
    ```javascript
    function show_custom_dialog() {
        let d = new frappe.ui.Dialog({
            title: 'Enter details',
            fields: [
                {
                    label: 'First Name',
                    fieldname: 'first_name',
                    fieldtype: 'Data'
                },
                {
                    label: 'Last Name',
                    fieldname: 'last_name',
                    fieldtype: 'Data'
                }
            ],
            primary_action_label: 'Submit',
            primary_action(values) {
                // Handle submission
                d.hide();
            }
        });
        
        d.show();
    }
    ```

18. **Q: How do you implement custom list views?**
    ```javascript
    frappe.listview_settings['DocType'] = {
        add_fields: ['status', 'priority'],
        get_indicator: function(doc) {
            if (doc.status === "Completed") {
                return [__("Completed"), "green", "status,=,Completed"];
            }
        },
        onload: function(listview) {
            // Custom list view logic
        }
    };
    ```

19. **Q: How do you implement custom workspaces?**
    ```python
    # workspace.py
    def get_data():
        return {
            'cards': [
                {
                    'label': 'Sales',
                    'items': [
                        {
                            'type': 'doctype',
                            'name': 'Sales Order',
                            'label': 'Sales Orders'
                        }
                    ]
                }
            ],
            'charts': [
                {
                    'name': 'Sales Analytics',
                    'chart_name': 'Sales Trends'
                }
            ]
        }
    ```

20. **Q: How do you implement custom reports?**
    ```python
    from frappe import _
    
    def execute(filters=None):
        columns = get_columns()
        data = get_data(filters)
        return columns, data
    
    def get_columns():
        return [
            {
                "fieldname": "name",
                "label": _("Name"),
                "fieldtype": "Link",
                "options": "DocType",
                "width": 140
            }
        ]
    ```

## Reports and Queries

21. **Q: How do you implement custom query reports?**
    ```python
    # query_report.py
    def get_data(filters):
        return frappe.db.sql("""
            SELECT 
                name, customer_name, grand_total
            FROM 
                `tabSales Invoice`
            WHERE 
                posting_date BETWEEN %(from_date)s AND %(to_date)s
        """, filters, as_dict=1)
    ```

22. **Q: How do you implement script reports?**
    ```python
    # script_report.py
    class CustomReport(object):
        def __init__(self, filters=None):
            self.filters = filters
        
        def run(self):
            columns = self.get_columns()
            data = self.get_data()
            return columns, data
        
        def get_columns(self):
            return [
                {"label": "ID", "fieldname": "id", "width": 100},
                {"label": "Name", "fieldname": "name", "width": 200}
            ]
    ```

23. **Q: How do you implement custom filters?**
    ```javascript
    // Custom filter
    frappe.query_reports["Custom Report"] = {
        "filters": [
            {
                "fieldname": "from_date",
                "label": __("From Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.add_months(
                    frappe.datetime.get_today(), -1
                ),
                "reqd": 1
            },
            {
                "fieldname": "to_date",
                "label": __("To Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.get_today(),
                "reqd": 1
            }
        ]
    };
    ```

24. **Q: How do you implement custom queries?**
    ```python
    # Custom database queries
    def get_custom_data():
        return frappe.db.sql("""
            SELECT 
                t1.name, 
                t1.customer_name,
                t2.item_code,
                t2.qty
            FROM 
                `tabSales Order` t1
                LEFT JOIN `tabSales Order Item` t2 
                ON t1.name = t2.parent
            WHERE 
                t1.docstatus = 1
        """, as_dict=True)
    ```

25. **Q: How do you implement report charts?**
    ```python
    def get_chart_data(data):
        labels = []
        values = []
        
        for d in data:
            labels.append(d.get("month"))
            values.append(d.get("amount"))
        
        return {
            "data": {
                "labels": labels,
                "datasets": [
                    {
                        "name": "Monthly Sales",
                        "values": values
                    }
                ]
            },
            "type": "line"
        }
    ```

## Customization and Extensions

26. **Q: How do you implement custom apps?**
    ```python
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    app_include_css = [
        "/assets/custom_app/css/custom.css"
    ]
    ```

27. **Q: How do you implement custom fields programmatically?**
    ```python
    def create_custom_fields():
        custom_fields = {
            "Customer": [
                {
                    "fieldname": "custom_field",
                    "label": "Custom Field",
                    "fieldtype": "Data",
                    "insert_after": "customer_name"
                }
            ]
        }
        
        for doctype, fields in custom_fields.items():
            for field in fields:
                if not frappe.db.exists(
                    "Custom Field", 
                    {"dt": doctype, "fieldname": field["fieldname"]}
                ):
                    frappe.get_doc({
                        "doctype": "Custom Field",
                        "dt": doctype,
                        **field
                    }).insert()
    ```

28. **Q: How do you implement custom roles and permissions?**
    ```python
    def create_custom_role():
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Role"
            role.desk_access = 1
            role.insert()
    
    def set_custom_permissions():
        # Custom DocPerm
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.read = 1
        docperm.write = 1
        docperm.create = 1
        docperm.submit = 0
        docperm.cancel = 0
        docperm.delete = 0
        docperm.insert()
    ```

29. **Q: How do you implement custom print formats?**
    ```html
    <!-- custom_format.html -->
    <div class="print-format">
        <div class="row">
            <div class="col-xs-6">
                <div class="row">
                    <div class="col-xs-5">
                        <img src="{{ letter_head }}">
                    </div>
                </div>
            </div>
        </div>
        
        <div class="row">
            <div class="col-xs-12">
                <div class="row">
                    <div class="col-xs-6">
                        <strong>{{ doc.customer_name }}</strong><br>
                        {{ doc.address_display }}
                    </div>
                </div>
            </div>
        </div>
    </div>
    ```

30. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.new_doc("Workflow")
        workflow.workflow_name = "Custom Workflow"
        workflow.document_type = "Custom DocType"
        workflow.workflow_state_field = "workflow_state"
        
        # States
        workflow.states = [
            {
                "state": "Draft",
                "doc_status": 0,
                "allow_edit": "Custom Role"
            },
            {
                "state": "Submitted",
                "doc_status": 1,
                "allow_edit": "System Manager"
            }
        ]
        
        # Transitions
        workflow.transitions = [
            {
                "state": "Draft",
                "action": "Submit",
                "next_state": "Submitted",
                "allowed": "Custom Role"
            }
        ]
        
        workflow.insert()
    ```

## Workflow and Automation

31. **Q: How do you implement custom workflows with multiple states?**
    ```python
    def setup_workflow():
        workflow = {
            "name": "Custom Process",
            "document_type": "Custom DocType",
            "states": [
                {
                    "state": "Draft",
                    "style": "Primary",
                    "doc_status": 0
                },
                {
                    "state": "Pending Approval",
                    "style": "Warning",
                    "doc_status": 0
                },
                {
                    "state": "Approved",
                    "style": "Success",
                    "doc_status": 1
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Custom User"
                },
                {
                    "state": "Pending Approval",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "Custom Approver"
                }
            ]
        }
        
        if not frappe.db.exists("Workflow", workflow["name"]):
            doc = frappe.get_doc({"doctype": "Workflow", **workflow})
            doc.insert()
    ```

32. **Q: How do you implement scheduled tasks?**
    ```python
    # hooks.py
    scheduler_events = {
        "daily": [
            "custom_app.tasks.daily_cleanup"
        ],
        "hourly": [
            "custom_app.tasks.send_reminders"
        ],
        "weekly": [
            "custom_app.tasks.weekly_report"
        ]
    }
    
    # tasks.py
    def daily_cleanup():
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    
    def send_reminders():
        # Send reminder emails
        pass
    ```

33. **Q: How do you implement document event handlers?**
    ```python
    # hooks.py
    doc_events = {
        "Sales Order": {
            "on_submit": "custom_app.events.so_on_submit",
            "on_cancel": "custom_app.events.so_on_cancel",
            "after_insert": "custom_app.events.so_after_insert"
        }
    }
    
    # events.py
    def so_on_submit(doc, method):
        # Create related documents
        create_project(doc)
        notify_departments(doc)
    
    def notify_departments(doc):
        frappe.publish_realtime(
            'sales_order_submitted',
            {"sales_order": doc.name}
        )
    ```

34. **Q: How do you implement custom notifications?**
    ```python
    def setup_notification():
        notification = frappe.new_doc("Notification")
        notification.name = "Custom Alert"
        notification.subject = "New {{ doc.doctype }} {{ doc.name }}"
        notification.document_type = "Custom DocType"
        notification.event = "Submit"
        notification.channel = "Email"
        notification.message = """
            Dear {{ doc.customer_name }},
            Your {{ doc.doctype }} {{ doc.name }} has been submitted.
        """
        notification.insert()
    ```

35. **Q: How do you implement email alerts?**
    ```python
    def send_custom_email(doc):
        frappe.sendmail(
            recipients=[doc.email],
            subject=f"Document {doc.name} Status Update",
            template="custom_email",
            args={
                "name": doc.name,
                "status": doc.status,
                "comments": doc.comments
            },
            header=["Status Update", "green"]
        )
    
    # custom_email.html
    <h3>Status Update</h3>
    <p>Document {{ name }} status has been updated to {{ status }}</p>
    {% if comments %}
    <p>Comments: {{ comments }}</p>
    {% endif %}
    ```

## Security and Permissions

36. **Q: How do you implement role-based access control?**
    ```python
    def setup_roles_and_permissions():
        # Create custom role
        if not frappe.db.exists("Role", "Custom Manager"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Manager"
            role.desk_access = 1
            role.insert()
        
        # Set permissions
        custom_manager_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        add_permission(
            "Custom DocType",
            "Custom Manager",
            custom_manager_perms
        )
    ```

37. **Q: How do you implement custom authentication?**
    ```python
    from frappe.auth import LoginManager
    
    class CustomLoginManager(LoginManager):
        def validate_user(self):
            # Custom validation logic
            super().validate_user()
            
            if not self.user_in_approved_list():
                frappe.throw("User not approved")
        
        def user_in_approved_list(self):
            return frappe.db.exists(
                "Approved Users",
                {"user": self.user}
            )
    ```

38. **Q: How do you implement field-level permissions?**
    ```python
    def setup_field_permissions():
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.permlevel = 1
        docperm.read = 1
        docperm.write = 0
        
        # Field permission in DocType
        custom_field = {
            "fieldname": "sensitive_data",
            "permlevel": 1
        }
        
        update_field_permission(
            "Custom DocType",
            custom_field
        )
    ```

39. **Q: How do you implement API authentication?**
    ```python
    @frappe.whitelist()
    def custom_api():
        # Check if user has permission
        if not frappe.has_permission("Custom DocType"):
            frappe.throw("Not permitted")
        
        # API token validation
        api_token = frappe.get_request_header("Authorization")
        if not validate_api_token(api_token):
            frappe.throw("Invalid API token")
        
        return "API response"
    
    def validate_api_token(token):
        return frappe.db.exists(
            "API Token",
            {"token": token, "enabled": 1}
        )
    ```

40. **Q: How do you implement user permissions?**
    ```python
    def setup_user_permissions():
        # Add user permission
        user_permission = frappe.new_doc("User Permission")
        user_permission.user = "user@example.com"
        user_permission.allow = "Custom DocType"
        user_permission.for_value = "CUST-001"
        user_permission.apply_to_all_doctypes = 1
        user_permission.insert()
        
        # Check user permission
        def validate_user_permission(doc, method):
            if not frappe.has_permission(
                doc.doctype,
                "write",
                doc
            ):
                frappe.throw("Not permitted")
    ```

## Deployment and Performance

41. **Q: How do you implement custom deployment configurations?**
    ```python
    # config.py
    from frappe.utils import cstr, cint
    
    def get_site_config():
        return {
            'maintenance_mode': 0,
            'pause_scheduler': 0,
            'developer_mode': 0,
            'disable_website_cache': 0,
            'rate_limit': {
                'limit': 100,
                'window': 3600
            }
        }
    ```

42. **Q: How do you implement caching strategies?**
    ```python
    from frappe.utils.caching import redis_cache
    
    @redis_cache
    def get_cached_data(key):
        # Expensive operation
        result = frappe.db.sql("""
            SELECT * FROM `tabCustom DocType`
            WHERE status = 'Active'
        """, as_dict=True)
        return result
    
    def clear_cache():
        frappe.cache().delete_key('cached_data')
    ```

43. **Q: How do you implement database optimization?**
    ```python
    def optimize_database():
        # Add indexes
        frappe.db.sql("""
            CREATE INDEX IF NOT EXISTS idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize tables
        frappe.db.sql("""
            OPTIMIZE TABLE `tabCustom DocType`
        """)
        
        # Clear unused data
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

44. **Q: How do you implement load balancing?**
    ```python
    # nginx configuration
    def get_nginx_config():
        return """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
        """
    ```

45. **Q: How do you implement monitoring and logging?**
    ```python
    import logging
    
    def setup_logging():
        # Custom logger
        logger = logging.getLogger("custom_app")
        logger.setLevel(logging.DEBUG)
        
        # File handler
        fh = logging.FileHandler("custom_app.log")
        fh.setLevel(logging.DEBUG)
        
        # Formatter
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        fh.setFormatter(formatter)
        logger.addHandler(fh)
        
        return logger
    
    # Usage
    logger = setup_logging()
    logger.info("Custom operation completed")
    ```

## Integration and Development

46. **Q: How do you implement external API integration?**
    ```python
    import requests
    
    class ExternalAPIIntegration:
        def __init__(self):
            self.api_key = frappe.conf.get("external_api_key")
            self.base_url = "https://api.external.com/v1"
        
        def make_request(self, endpoint, method="GET", data=None):
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            response = requests.request(
                method,
                f"{self.base_url}/{endpoint}",
                headers=headers,
                json=data
            )
            
            response.raise_for_status()
            return response.json()
    ```

47. **Q: How do you implement webhooks?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def webhook_handler():
        if not validate_webhook_signature():
            frappe.throw("Invalid webhook signature")
        
        data = frappe.request.get_json()
        
        # Process webhook data
        process_webhook_data(data)
        
        return "Success"
    
    def validate_webhook_signature():
        signature = frappe.get_request_header("X-Webhook-Signature")
        # Validate signature
        return True
    ```

48. **Q: How do you implement custom REST endpoints?**
    ```python
    @frappe.whitelist()
    def custom_endpoint():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

49. **Q: How do you implement file handling?**
    ```python
    def handle_file_upload():
        file_doc = frappe.get_doc({
            "doctype": "File",
            "attached_to_doctype": "Custom DocType",
            "attached_to_name": "DOC-001",
            "attached_to_field": "attachment",
            "file_url": "",
            "file_name": "document.pdf",
            "is_private": 1,
            "content": frappe.request.files['file']
        }).insert()
        
        return file_doc
    ```

50. **Q: How do you implement custom apps and modules?**
    ```python
    # modules.txt
    Custom App
    Core
    Integration
    
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    ```

## ERPNext Integration

51. **Q: How do you extend ERPNext DocTypes?**
    ```python
    # custom_sales_order.py
    from erpnext.selling.doctype.sales_order.sales_order import SalesOrder
    
    class CustomSalesOrder(SalesOrder):
        def validate(self):
            super().validate()
            self.validate_custom_fields()
        
        def validate_custom_fields(self):
            if self.custom_field < 0:
                frappe.throw("Custom field cannot be negative")
    ```

52. **Q: How do you implement custom pricing rules?**
    ```python
    def get_custom_price(item_code, customer):
        pricing_rule = frappe.get_doc({
            "doctype": "Pricing Rule",
            "title": f"Custom Price for {customer}",
            "apply_on": "Item Code",
            "items": [{
                "item_code": item_code
            }],
            "selling": 1,
            "rate_or_discount": "Rate",
            "rate": 100
        })
        return pricing_rule
    ```

53. **Q: How do you implement custom workflows in ERPNext?**
    ```python
    def create_custom_workflow():
        workflow = {
            "name": "Custom SO Workflow",
            "document_type": "Sales Order",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Sales User"
                },
                {
                    "state": "Pending Approval",
                    "doc_status": 0,
                    "allow_edit": "Sales Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Sales User"
                }
            ]
        }
        return workflow
    ```

54. **Q: How do you implement custom reports in ERPNext?**
    ```python
    def get_custom_report():
        return {
            "columns": [
                {
                    "fieldname": "customer",
                    "label": "Customer",
                    "fieldtype": "Link",
                    "options": "Customer",
                    "width": 180
                },
                {
                    "fieldname": "total_sales",
                    "label": "Total Sales",
                    "fieldtype": "Currency",
                    "width": 140
                }
            ],
            "filters": [
                {
                    "fieldname": "from_date",
                    "label": "From Date",
                    "fieldtype": "Date",
                    "default": frappe.utils.add_months(
                        frappe.utils.nowdate(), -1
                    )
                }
            ]
        }
    ```

55. **Q: How do you implement custom item variants?**
    ```python
    def create_item_variant():
        template = frappe.get_doc("Item", "ITEM-TEMP-001")
        variant = frappe.get_doc({
            "doctype": "Item",
            "variant_of": template.name,
            "item_code": "ITEM-VAR-001",
            "item_name": "Custom Variant",
            "item_group": template.item_group,
            "stock_uom": template.stock_uom,
            "attributes": [
                {
                    "attribute": "Size",
                    "attribute_value": "Large"
                }
            ]
        }).insert()
        return variant
    ```

## Advanced Development

56. **Q: How do you implement custom web pages?**
    ```python
    # hooks.py
    website_route_rules = [
        {"from_route": "/custom", "to_route": "custom/index"}
    ]
    
    # custom/index.html
    {% extends "templates/web.html" %}
    
    {% block page_content %}
    <div class="custom-page">
        <h1>{{ title }}</h1>
        {{ content }}
    </div>
    {% endblock %}
    
    # custom/index.py
    def get_context(context):
        context.title = "Custom Page"
        context.content = frappe.get_doc(
            "Custom Content", 
            "CONTENT-001"
        ).content
    ```

57. **Q: How do you implement custom portals?**
    ```python
    # hooks.py
    portal_menu_items = [
        {
            "title": "Custom Portal",
            "route": "/custom-portal",
            "reference_doctype": "Custom DocType",
            "role": "Customer"
        }
    ]
    
    # custom_portal.js
    frappe.pages['custom-portal'].on_page_load = function(wrapper) {
        var page = frappe.ui.make_app_page({
            parent: wrapper,
            title: 'Custom Portal',
            single_column: true
        });
        
        page.add_inner_button(__('New'), function() {
            frappe.new_doc('Custom DocType');
        });
    }
    ```

58. **Q: How do you implement custom dashboards?**
    ```python
    def get_dashboard_data(data):
        return {
            'fieldname': 'custom_doctype',
            'transactions': [
                {
                    'label': _('Related Documents'),
                    'items': ['Sales Order', 'Purchase Order']
                }
            ],
            'charts': [
                {
                    'name': 'Custom Chart',
                    'chart_name': 'Monthly Trends',
                    'chart_type': 'Line'
                }
            ]
        }
    ```

59. **Q: How do you implement custom scripts?**
    ```python
    # Server Script
    doc = frappe.get_doc({
        "doctype": "Server Script",
        "name": "Custom Server Script",
        "script_type": "DocType Event",
        "doc_type": "Custom DocType",
        "event": "Before Save",
        "script": """
    if doc.status == "Completed" and not doc.completion_date:
        doc.completion_date = frappe.utils.nowdate()
        """
    })
    
    # Client Script
    doc = frappe.get_doc({
        "doctype": "Client Script",
        "name": "Custom Client Script",
        "dt": "Custom DocType",
        "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
        """
    })
    ```

60. **Q: How do you implement custom APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    
    def process_request(params):
        # Custom processing logic
        return {
            "processed": True,
            "params": params
        }
    ```

## Advanced Features

61. **Q: How do you implement custom email templates?**
    ```python
    def create_email_template():
        template = frappe.get_doc({
            "doctype": "Email Template",
            "name": "Custom Template",
            "subject": "{{ doc.name }} - Status Update",
            "response": """
                Dear {{ doc.customer_name }},
                
                Your {{ doc.doctype }} ({{ doc.name }}) 
                has been updated to {{ doc.status }}.
                
                Regards,
                {{ frappe.defaults.get_global_default('company') }}
            """
        }).insert()
        return template
    ```

62. **Q: How do you implement custom print formats?**
    ```python
    def create_print_format():
        format_doc = frappe.get_doc({
            "doctype": "Print Format",
            "name": "Custom Format",
            "doc_type": "Custom DocType",
            "standard": "No",
            "print_format_type": "Jinja",
            "html": """
                <div class="print-heading">
                    <h2>{{ doc.name }}</h2>
                </div>
                
                <div class="row">
                    <div class="col-xs-6">
                        <div class="row">
                            <div class="col-xs-5">
                                <label>Customer</label>
                            </div>
                            <div class="col-xs-7">
                                {{ doc.customer_name }}
                            </div>
                        </div>
                    </div>
                </div>
            """
        }).insert()
        return format_doc
    ```

63. **Q: How do you implement custom web forms?**
    ```python
    def create_web_form():
        web_form = frappe.get_doc({
            "doctype": "Web Form",
            "title": "Custom Form",
            "route": "custom-form",
            "doc_type": "Custom DocType",
            "web_form_fields": [
                {
                    "fieldname": "customer_name",
                    "fieldtype": "Data",
                    "label": "Customer Name",
                    "reqd": 1
                },
                {
                    "fieldname": "email",
                    "fieldtype": "Data",
                    "label": "Email",
                    "reqd": 1,
                    "options": "Email"
                }
            ]
        }).insert()
        return web_form
    ```

64. **Q: How do you implement custom roles and permissions?**
    ```python
    def setup_custom_role():
        # Create role
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.get_doc({
                "doctype": "Role",
                "role_name": "Custom Role",
                "desk_access": 1
            }).insert()
        
        # Set permissions
        custom_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        if not frappe.db.exists(
            "Custom DocPerm",
            {
                "parent": "Custom DocType",
                "role": "Custom Role"
            }
        ):
            perm = frappe.get_doc({
                "doctype": "Custom DocPerm",
                "parent": "Custom DocType",
                "parenttype": "DocType",
                "parentfield": "permissions",
                "role": "Custom Role",
                **custom_perms
            }).insert()
    ```

65. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.get_doc({
            "doctype": "Workflow",
            "workflow_name": "Custom Process",
            "document_type": "Custom DocType",
            "workflow_state_field": "workflow_state",
            "states": [
                {
                    "state": "Draft",
                    "doc_status": 0,
                    "allow_edit": "Custom Role"
                },
                {
                    "state": "Pending",
                    "doc_status": 0,
                    "allow_edit": "System Manager"
                },
                {
                    "state": "Approved",
                    "doc_status": 1,
                    "allow_edit": "System Manager"
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit",
                    "next_state": "Pending",
                    "allowed": "Custom Role"
                },
                {
                    "state": "Pending",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "System Manager"
                }
            ]
        }).insert()
        return workflow
    ```

## Testing and Debugging

66. **Q: How do you implement unit tests?**
    ```python
    import unittest
    
    class TestCustomDoc(unittest.TestCase):
        def setUp(self):
            # Create test data
            self.doc = frappe.get_doc({
                "doctype": "Custom DocType",
                "title": "Test Doc"
            }).insert()
        
        def tearDown(self):
            # Cleanup test data
            frappe.delete_doc(
                "Custom DocType",
                self.doc.name
            )
        
        def test_validation(self):
            self.doc.status = "Completed"
            self.assertRaises(
                frappe.ValidationError,
                self.doc.save
            )
    ```

67. **Q: How do you implement integration tests?**
    ```python
    class TestIntegration(unittest.TestCase):
        def test_api_integration(self):
            # Test API endpoint
            response = frappe.call(
                'custom_app.api.custom_endpoint',
                param1="test",
                param2="value"
            )
            
            self.assertEqual(
                response.get("status"),
                "success"
            )
        
        def test_workflow(self):
            # Test workflow transitions
            doc = create_test_doc()
            doc.submit()
            
            self.assertEqual(
                doc.workflow_state,
                "Pending Approval"
            )
    ```

68. **Q: How do you implement custom error handling?**
    ```python
    class CustomError(Exception):
        def __init__(self, message, error_code=None):
            super().__init__(message)
            self.error_code = error_code
    
    def custom_error_handler():
        try:
            # Risky operation
            result = process_data()
            
            if not result:
                raise CustomError(
                    "Processing failed",
                    "PROC001"
                )
            
            return result
            
        except CustomError as e:
            frappe.log_error(
                title="Custom Error",
                message=str(e)
            )
            frappe.throw(
                str(e),
                exc=e.__class__
            )
    ```

69. **Q: How do you implement debugging tools?**
    ```python
    def debug_document(doc):
        # Log document state
        frappe.logger().debug({
            "doctype": doc.doctype,
            "name": doc.name,
            "status": doc.status,
            "modified": doc.modified
        })
        
        # Track changes
        frappe.logger().debug({
            "changed_fields": doc.get_changed(),
            "user": frappe.session.user,
            "timestamp": frappe.utils.now()
        })
    ```

70. **Q: How do you implement performance profiling?**
    ```python
    import cProfile
    import pstats
    
    def profile_function():
        profiler = cProfile.Profile()
        profiler.enable()
        
        # Function to profile
        result = expensive_operation()
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats()
        
        return result
    ```

## Advanced Customization

71. **Q: How do you implement custom field types?**
    ```python
    class CustomField(frappe.model.document.Document):
        def validate(self):
            if self.fieldtype == "Custom":
                self.validate_custom_field()
        
        def validate_custom_field(self):
            # Custom validation logic
            if not self.options:
                frappe.throw(
                    "Options required for Custom field type"
                )
    ```

72. **Q: How do you implement custom controllers?**
    ```python
    class CustomController(frappe.model.document.Document):
        def validate(self):
            self.validate_custom_logic()
        
        def on_submit(self):
            self.create_related_docs()
        
        def validate_custom_logic(self):
            if self.end_date <= self.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
        
        def create_related_docs(self):
            # Create related documents
            self.create_project()
            self.create_tasks()
    ```

73. **Q: How do you implement custom doctypes?**
    ```python
    def create_custom_doctype():
        doctype = frappe.get_doc({
            "doctype": "DocType",
            "name": "Custom DocType",
            "module": "Custom App",
            "custom": 1,
            "fields": [
                {
                    "fieldname": "title",
                    "fieldtype": "Data",
                    "label": "Title",
                    "reqd": 1
                },
                {
                    "fieldname": "status",
                    "fieldtype": "Select",
                    "label": "Status",
                    "options": "Draft\nPending\nCompleted"
                }
            ],
            "permissions": [
                {
                    "role": "System Manager",
                    "read": 1,
                    "write": 1,
                    "create": 1,
                    "delete": 1
                }
            ]
        }).insert()
        return doctype
    ```

74. **Q: How do you implement custom scripts?**
    ```python
    def create_custom_scripts():
        # Server script
        server_script = frappe.get_doc({
            "doctype": "Server Script",
            "name": "Custom Server Script",
            "script_type": "DocType Event",
            "doc_type": "Custom DocType",
            "event": "Before Save",
            "script": """
    if doc.status == "Completed":
        doc.completion_date = frappe.utils.now()
            """
        }).insert()
        
        # Client script
        client_script = frappe.get_doc({
            "doctype": "Client Script",
            "name": "Custom Client Script",
            "dt": "Custom DocType",
            "script": """
    frappe.ui.form.on('Custom DocType', {
        refresh: function(frm) {
            frm.add_custom_button('Process', function() {
                // Custom processing logic
            });
        }
    });
            """
        }).insert()
    ```

75. **Q: How do you implement custom reports?**
    ```python
    def create_custom_report():
        report = frappe.get_doc({
            "doctype": "Report",
            "name": "Custom Report",
            "ref_doctype": "Custom DocType",
            "report_type": "Script Report",
            "is_standard": "No",
            "module": "Custom App",
            "query": """
    SELECT
        name,
        creation,
        modified,
        status
    FROM
        `tabCustom DocType`
    WHERE
        status = %(status)s
            """,
            "filters": [
                {
                    "fieldname": "status",
                    "label": "Status",
                    "fieldtype": "Select",
                    "options": "Draft\nPending\nCompleted",
                    "default": "Pending"
                }
            ]
        }).insert()
        return report
    ```

## Data Management

76. **Q: How do you implement data import/export?**
    ```python
    def import_data():
        from frappe.core.doctype.data_import.data_import import (
            import_file
        )
        
        # Import data
        import_file(
            doctype="Custom DocType",
            file_path="path/to/import.csv",
            import_type="Insert",
            submit_after_import=True
        )
    
    def export_data():
        from frappe.desk.query_report import export_query
        
        # Export data
        export_query(
            "Custom Report",
            "CSV",
            filters={"status": "Completed"}
        )
    ```

77. **Q: How do you implement data migrations?**
    ```python
    def migrate_data():
        # Create migration plan
        plan = frappe.get_doc({
            "doctype": "Data Migration Plan",
            "name": "Custom Migration",
            "module": "Custom App",
            "mappings": [
                {
                    "mapping": "Customer to Contact",
                    "remote_objectname": "Contact",
                    "page_length": 10
                }
            ]
        })
        
        # Run migration
        run = frappe.get_doc({
            "doctype": "Data Migration Run",
            "data_migration_plan": plan.name
        }).insert()
        
        run.run()
    ```

78. **Q: How do you implement data archival?**
    ```python
    def archive_data():
        # Archive old records
        frappe.db.sql("""
            INSERT INTO `tabArchived DocType`
            SELECT *
            FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
        
        # Delete archived records
        frappe.db.sql("""
            DELETE FROM `tabCustom DocType`
            WHERE modified < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        """)
    ```

79. **Q: How do you implement data validation?**
    ```python
    def validate_data():
        def validate_custom_doc(doc, method):
            # Validate required fields
            if not doc.title:
                frappe.throw("Title is required")
            
            # Validate business logic
            if doc.end_date <= doc.start_date:
                frappe.throw(
                    "End date must be after start date"
                )
            
            # Validate uniqueness
            if frappe.db.exists(
                "Custom DocType",
                {
                    "title": doc.title,
                    "name": ("!=", doc.name)
                }
            ):
                frappe.throw("Title must be unique")
    ```

80. **Q: How do you implement data cleanup?**
    ```python
    def cleanup_data():
        # Delete temporary files
        frappe.db.sql("""
            DELETE FROM `tabFile`
            WHERE attached_to_doctype = 'Custom DocType'
            AND creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
        
        # Clear error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 7 DAY)
        """)
        
        # Remove orphaned records
        frappe.db.sql("""
            DELETE FROM `tabCustom Child Table`
            WHERE parent NOT IN (
                SELECT name
                FROM `tabCustom DocType`
            )
        """)
    ```

## System Administration

81. **Q: How do you implement system monitoring?**
    ```python
    def monitor_system():
        # Monitor database
        db_stats = frappe.db.sql("""
            SHOW STATUS
            WHERE Variable_name IN (
                'Threads_connected',
                'Questions',
                'Slow_queries'
            )
        """, as_dict=1)
        
        # Monitor background jobs
        jobs = frappe.db.sql("""
            SELECT status, COUNT(*) as count
            FROM `tabQueue Job`
            GROUP BY status
        """, as_dict=1)
        
        # Log statistics
        frappe.logger().info({
            "db_stats": db_stats,
            "background_jobs": jobs
        })
    ```

82. **Q: How do you implement backup and recovery?**
    ```python
    def manage_backups():
        from frappe.utils.backups import backup
        
        # Create backup
        backup_path = backup(
            with_files=True,
            backup_path_db="/path/to/db/backup",
            backup_path_files="/path/to/files/backup"
        )
        
        # Restore from backup
        def restore_backup():
            # Stop server
            os.system("bench stop")
            
            # Restore database
            os.system(
                f"mysql -u {db_name} < {backup_path}/database.sql"
            )
            
            # Restore files
            os.system(
                f"rsync -a {backup_path}/files/ ./sites/site1.local/public/files/"
            )
            
            # Start server
            os.system("bench start")
    ```

83. **Q: How do you implement system updates?**
    ```python
    def update_system():
        # Update apps
        os.system("bench update")
        
        # Update dependencies
        os.system("bench setup requirements")
        
        # Migrate database
        os.system("bench migrate")
        
        # Clear cache
        frappe.clear_cache()
        
        # Restart server
        os.system("bench restart")
    ```

84. **Q: How do you implement system maintenance?**
    ```python
    def maintain_system():
        # Cleanup temporary files
        cleanup_temporary_files()
        
        # Optimize database
        frappe.db.sql("OPTIMIZE TABLE `tabCustom DocType`")
        
        # Clear cache
        frappe.clear_cache()
        
        # Delete old error logs
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

85. **Q: How do you implement system security?**
    ```python
    def secure_system():
        # Set secure configuration
        frappe.db.set_value(
            "System Settings",
            None,
            {
                "allow_login_using_mobile_number": 0,
                "allow_login_using_user_name": 0,
                "allow_guests_to_upload_files": 0,
                "force_user_to_reset_password": 1
            }
        )
        
        # Enable two-factor authentication
        frappe.db.set_value(
            "System Settings",
            None,
            "enable_two_factor_auth",
            1
        )
    ```

## Performance Optimization

86. **Q: How do you implement caching strategies?**
    ```python
    def implement_caching():
        # Redis cache
        @frappe.whitelist()
        def get_cached_data():
            cache_key = "custom_data"
            data = frappe.cache().get_value(cache_key)
            
            if not data:
                data = get_expensive_data()
                frappe.cache().set_value(
                    cache_key,
                    data,
                    expires_in_sec=3600
                )
            
            return data
    ```

87. **Q: How do you implement query optimization?**
    ```python
    def optimize_queries():
        # Use proper indexes
        frappe.db.sql("""
            CREATE INDEX idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize joins
        results = frappe.db.sql("""
            SELECT t1.name, t2.item_code
            FROM `tabCustom DocType` t1
            INNER JOIN `tabCustom Child Table` t2
            ON t1.name = t2.parent
            WHERE t1.status = 'Active'
        """, as_dict=1)
    ```

88. **Q: How do you implement background jobs?**
    ```python
    def manage_background_jobs():
        # Enqueue long-running job
        frappe.enqueue(
            'custom_app.tasks.process_data',
            queue='long',
            timeout=1500,
            doc_name="DOC-001"
        )
        
        # Monitor job status
        def get_job_status(job_id):
            return frappe.db.get_value(
                'Queue Job',
                {'job_id': job_id},
                'status'
            )
    ```

89. **Q: How do you implement load balancing?**
    ```python
    def configure_load_balancing():
        # Nginx configuration
        nginx_config = """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
            }
        }
        """
    ```

90. **Q: How do you implement performance monitoring?**
    ```python
    def monitor_performance():
        import time
        
        def log_slow_queries():
            start_time = time.time()
            
            # Execute query
            result = frappe.db.sql(
                "SELECT * FROM `tabCustom DocType`"
            )
            
            execution_time = time.time() - start_time
            
            if execution_time > 1.0:  # Log if > 1 second
                frappe.logger().warning(
                    f"Slow query detected: {execution_time}s"
                )
    ```

## API Development

91. **Q: How do you implement REST APIs?**
    ```python
    @frappe.whitelist()
    def custom_api():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

92. **Q: How do you implement API authentication?**
    ```python
    def authenticate_api():
        def validate_api_key():
            api_key = frappe.get_request_header(
                "Authorization"
            )
            
            if not api_key:
                frappe.throw(
                    "API key required",
                    frappe.AuthenticationError
                )
            
            if not frappe.db.exists(
                "API Key",
                {
                    "api_key": api_key,
                    "enabled": 1
                }
            ):
                frappe.throw(
                    "Invalid API key",
                    frappe.AuthenticationError
                )
    ```

93. **Q: How do you implement API rate limiting?**
    ```python
    def implement_rate_limiting():
        def check_rate_limit():
            user = frappe.session.user
            cache_key = f"rate_limit:{user}"
            
            # Get current count
            count = frappe.cache().get_value(cache_key) or 0
            
            if count >=
# 100 Advanced Frappe Framework Interview Questions and Answers

## Frappe Basics and Architecture

1. **Q: What is Frappe Framework and what are its key features?**
   - A: Frappe is a full-stack web application framework built on Python and JavaScript. Key features include:
   - Meta-data driven architecture
   - Built-in ORM
   - Role-based permissions
   - REST API
   - Real-time updates
   - Extensible architecture
   - Document versioning
   - Background jobs
   - Built-in web forms
   - Multi-tenancy support

2. **Q: How does Frappe's architecture work?**
   ```python
   # Example of Frappe's MVC-like architecture
   # Model (DocType)
   class Customer(Document):
       def validate(self):
           self.validate_credit_limit()
   
   # Controller (Python API)
   @frappe.whitelist()
   def get_customer_details(customer_id):
       return frappe.get_doc("Customer", customer_id)
   
   # View (Form/List)
   # customer.js
   frappe.ui.form.on('Customer', {
       refresh: function(frm) {
           // Form rendering logic
       }
   })
   ```

3. **Q: Explain Frappe's database structure.**
   - A: Frappe uses MariaDB/MySQL with key components:
   ```sql
   -- Main DocType table
   CREATE TABLE `tabCustomer` (
       `name` varchar(140) PRIMARY KEY,
       `creation` datetime,
       `modified` datetime,
       `modified_by` varchar(140),
       `owner` varchar(140),
       `docstatus` int(1) DEFAULT 0,
       -- Custom fields follow
   );
   
   -- Single DocType
   CREATE TABLE `tabSingles` (
       `doctype` varchar(140),
       `field` varchar(140),
       `value` text
   );
   ```

4. **Q: How do you implement custom apps in Frappe?**
   ```bash
   # Creating new app
   bench new-app my_custom_app
   
   # App structure
   my_custom_app/
   ├── MANIFEST.in
   ├── README.md
   ├── my_custom_app/
   │   ├── __init__.py
   │   ├── hooks.py
   │   ├── modules.txt
   │   ├── patches.txt
   │   └── templates/
   └── setup.py
   ```

5. **Q: How do you handle hooks in Frappe?**
   ```python
   # hooks.py
   app_name = "my_custom_app"
   app_title = "My Custom App"
   
   doc_events = {
       "Sales Order": {
           "on_submit": "my_custom_app.events.so_on_submit",
           "on_cancel": "my_custom_app.events.so_on_cancel"
       }
   }
   
   scheduler_events = {
       "daily": [
           "my_custom_app.tasks.daily_cleanup"
       ]
   }
   ```

## DocTypes and Forms

6. **Q: How do you create custom DocTypes?**
   ```json
   {
       "name": "Custom DocType",
       "doctype": "DocType",
       "module": "My Custom App",
       "custom": 1,
       "fields": [
           {
               "fieldname": "title",
               "fieldtype": "Data",
               "label": "Title",
               "reqd": 1
           }
       ],
       "permissions": [
           {
               "role": "System Manager",
               "read": 1,
               "write": 1,
               "create": 1,
               "delete": 1
           }
       ]
   }
   ```

7. **Q: How do you implement custom form validations?**
   ```python
   # Python controller
   class CustomDoc(Document):
       def validate(self):
           if self.end_date < self.start_date:
               frappe.throw("End date cannot be before start date")
   
   # Client-side validation
   frappe.ui.form.on('Custom Doc', {
       validate: function(frm) {
           if (frm.doc.amount < 0) {
               frappe.throw(__("Amount cannot be negative"));
           }
       }
   });
   ```

8. **Q: How do you implement custom fields?**
   ```python
   # Adding custom field programmatically
   def create_custom_field():
       custom_field = frappe.get_doc({
           "doctype": "Custom Field",
           "dt": "Customer",
           "fieldname": "custom_field",
           "fieldtype": "Data",
           "label": "Custom Field",
           "insert_after": "customer_name"
       }).insert()
   ```

9. **Q: How do you implement child tables?**
   ```json
   {
       "name": "Parent DocType",
       "doctype": "DocType",
       "fields": [
           {
               "fieldname": "items",
               "fieldtype": "Table",
               "label": "Items",
               "options": "Child DocType"
           }
       ]
   }
   ```

10. **Q: How do you implement form scripts?**
    ```javascript
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            frm.add_custom_button(__('Custom Action'), function() {
                // Custom action logic
            });
        },
        
        validate: function(frm) {
            // Validation logic
        },
        
        before_save: function(frm) {
            // Pre-save logic
        }
    });
    ```

## Server Scripts and API

11. **Q: How do you implement server-side scripts?**
    ```python
    # Server script for custom validation
    def validate(doc, method):
        if doc.status == "Completed" and not doc.completion_date:
            frappe.throw("Completion date is mandatory")
    
    # Custom API method
    @frappe.whitelist()
    def custom_method(param1, param2):
        # Method logic
        return {"status": "success"}
    ```

12. **Q: How do you implement custom REST APIs?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def get_data():
        return frappe.get_list("DocType",
            fields=["name", "creation"],
            filters={"published": 1}
        )
    
    # Custom API endpoint
    @frappe.whitelist()
    def update_status(docname, status):
        doc = frappe.get_doc("DocType", docname)
        doc.status = status
        doc.save()
        return doc
    ```

13. **Q: How do you implement background jobs?**
    ```python
    # Queuing a background job
    def enqueue_job():
        frappe.enqueue(
            'my_custom_app.tasks.long_running_task',
            queue='long',
            timeout=300,
            doc_name=self.name
        )
    
    # Background job implementation
    def long_running_task(doc_name):
        # Long running process
        frappe.publish_realtime(
            'task_progress',
            {'progress': 50, 'doc_name': doc_name}
        )
    ```

14. **Q: How do you implement custom commands?**
    ```python
    # Custom bench command
    import click
    from frappe.commands import pass_context
    
    @click.command('custom-command')
    @pass_context
    def custom_command(context):
        """Custom command description"""
        frappe.init(site=context.sites[0])
        frappe.connect()
        # Command logic
        frappe.destroy()
    
    commands = [custom_command]
    ```

15. **Q: How do you implement document events?**
    ```python
    # Document event handlers
    def on_update(doc, method):
        frappe.msgprint(f"Document {doc.name} updated")
    
    def on_trash(doc, method):
        # Cleanup related records
        frappe.delete_doc("Related DocType", 
                         filters={"parent": doc.name})
    ```

## Client Scripts and Events

16. **Q: How do you implement client-side scripting?**
    ```javascript
    // Client script
    frappe.ui.form.on('DocType', {
        refresh: function(frm) {
            // Add custom button
            frm.add_custom_button(__('Process'), function() {
                process_document(frm);
            });
        },
        
        custom_field: function(frm) {
            // Field change handler
            calculate_totals(frm);
        }
    });
    
    function process_document(frm) {
        frappe.call({
            method: 'my_custom_app.api.process_doc',
            args: {
                docname: frm.doc.name
            },
            callback: function(r) {
                frappe.msgprint(r.message);
            }
        });
    }
    ```

17. **Q: How do you implement custom dialogs?**
    ```javascript
    function show_custom_dialog() {
        let d = new frappe.ui.Dialog({
            title: 'Enter details',
            fields: [
                {
                    label: 'First Name',
                    fieldname: 'first_name',
                    fieldtype: 'Data'
                },
                {
                    label: 'Last Name',
                    fieldname: 'last_name',
                    fieldtype: 'Data'
                }
            ],
            primary_action_label: 'Submit',
            primary_action(values) {
                // Handle submission
                d.hide();
            }
        });
        
        d.show();
    }
    ```

18. **Q: How do you implement custom list views?**
    ```javascript
    frappe.listview_settings['DocType'] = {
        add_fields: ['status', 'priority'],
        get_indicator: function(doc) {
            if (doc.status === "Completed") {
                return [__("Completed"), "green", "status,=,Completed"];
            }
        },
        onload: function(listview) {
            // Custom list view logic
        }
    };
    ```

19. **Q: How do you implement custom workspaces?**
    ```python
    # workspace.py
    def get_data():
        return {
            'cards': [
                {
                    'label': 'Sales',
                    'items': [
                        {
                            'type': 'doctype',
                            'name': 'Sales Order',
                            'label': 'Sales Orders'
                        }
                    ]
                }
            ],
            'charts': [
                {
                    'name': 'Sales Analytics',
                    'chart_name': 'Sales Trends'
                }
            ]
        }
    ```

20. **Q: How do you implement custom reports?**
    ```python
    from frappe import _
    
    def execute(filters=None):
        columns = get_columns()
        data = get_data(filters)
        return columns, data
    
    def get_columns():
        return [
            {
                "fieldname": "name",
                "label": _("Name"),
                "fieldtype": "Link",
                "options": "DocType",
                "width": 140
            }
        ]
    ```

## Reports and Queries

21. **Q: How do you implement custom query reports?**
    ```python
    # query_report.py
    def get_data(filters):
        return frappe.db.sql("""
            SELECT 
                name, customer_name, grand_total
            FROM 
                `tabSales Invoice`
            WHERE 
                posting_date BETWEEN %(from_date)s AND %(to_date)s
        """, filters, as_dict=1)
    ```

22. **Q: How do you implement script reports?**
    ```python
    # script_report.py
    class CustomReport(object):
        def __init__(self, filters=None):
            self.filters = filters
        
        def run(self):
            columns = self.get_columns()
            data = self.get_data()
            return columns, data
        
        def get_columns(self):
            return [
                {"label": "ID", "fieldname": "id", "width": 100},
                {"label": "Name", "fieldname": "name", "width": 200}
            ]
    ```

23. **Q: How do you implement custom filters?**
    ```javascript
    // Custom filter
    frappe.query_reports["Custom Report"] = {
        "filters": [
            {
                "fieldname": "from_date",
                "label": __("From Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.add_months(
                    frappe.datetime.get_today(), -1
                ),
                "reqd": 1
            },
            {
                "fieldname": "to_date",
                "label": __("To Date"),
                "fieldtype": "Date",
                "default": frappe.datetime.get_today(),
                "reqd": 1
            }
        ]
    };
    ```

24. **Q: How do you implement custom queries?**
    ```python
    # Custom database queries
    def get_custom_data():
        return frappe.db.sql("""
            SELECT 
                t1.name, 
                t1.customer_name,
                t2.item_code,
                t2.qty
            FROM 
                `tabSales Order` t1
                LEFT JOIN `tabSales Order Item` t2 
                ON t1.name = t2.parent
            WHERE 
                t1.docstatus = 1
        """, as_dict=True)
    ```

25. **Q: How do you implement report charts?**
    ```python
    def get_chart_data(data):
        labels = []
        values = []
        
        for d in data:
            labels.append(d.get("month"))
            values.append(d.get("amount"))
        
        return {
            "data": {
                "labels": labels,
                "datasets": [
                    {
                        "name": "Monthly Sales",
                        "values": values
                    }
                ]
            },
            "type": "line"
        }
    ```

## Customization and Extensions

26. **Q: How do you implement custom apps?**
    ```python
    # hooks.py
    app_name = "custom_app"
    app_title = "Custom App"
    app_publisher = "Your Company"
    app_description = "Custom App Description"
    app_icon = "octicon octicon-file-directory"
    app_color = "grey"
    app_email = "your@email.com"
    app_license = "MIT"
    
    # Includes in <head>
    app_include_js = [
        "/assets/custom_app/js/custom.js"
    ]
    app_include_css = [
        "/assets/custom_app/css/custom.css"
    ]
    ```

27. **Q: How do you implement custom fields programmatically?**
    ```python
    def create_custom_fields():
        custom_fields = {
            "Customer": [
                {
                    "fieldname": "custom_field",
                    "label": "Custom Field",
                    "fieldtype": "Data",
                    "insert_after": "customer_name"
                }
            ]
        }
        
        for doctype, fields in custom_fields.items():
            for field in fields:
                if not frappe.db.exists(
                    "Custom Field", 
                    {"dt": doctype, "fieldname": field["fieldname"]}
                ):
                    frappe.get_doc({
                        "doctype": "Custom Field",
                        "dt": doctype,
                        **field
                    }).insert()
    ```

28. **Q: How do you implement custom roles and permissions?**
    ```python
    def create_custom_role():
        if not frappe.db.exists("Role", "Custom Role"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Role"
            role.desk_access = 1
            role.insert()
    
    def set_custom_permissions():
        # Custom DocPerm
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.read = 1
        docperm.write = 1
        docperm.create = 1
        docperm.submit = 0
        docperm.cancel = 0
        docperm.delete = 0
        docperm.insert()
    ```

29. **Q: How do you implement custom print formats?**
    ```html
    <!-- custom_format.html -->
    <div class="print-format">
        <div class="row">
            <div class="col-xs-6">
                <div class="row">
                    <div class="col-xs-5">
                        <img src="{{ letter_head }}">
                    </div>
                </div>
            </div>
        </div>
        
        <div class="row">
            <div class="col-xs-12">
                <div class="row">
                    <div class="col-xs-6">
                        <strong>{{ doc.customer_name }}</strong><br>
                        {{ doc.address_display }}
                    </div>
                </div>
            </div>
        </div>
    </div>
    ```

30. **Q: How do you implement custom workflows?**
    ```python
    def create_custom_workflow():
        workflow = frappe.new_doc("Workflow")
        workflow.workflow_name = "Custom Workflow"
        workflow.document_type = "Custom DocType"
        workflow.workflow_state_field = "workflow_state"
        
        # States
        workflow.states = [
            {
                "state": "Draft",
                "doc_status": 0,
                "allow_edit": "Custom Role"
            },
            {
                "state": "Submitted",
                "doc_status": 1,
                "allow_edit": "System Manager"
            }
        ]
        
        # Transitions
        workflow.transitions = [
            {
                "state": "Draft",
                "action": "Submit",
                "next_state": "Submitted",
                "allowed": "Custom Role"
            }
        ]
        
        workflow.insert()
    ```

## Workflow and Automation

31. **Q: How do you implement custom workflows with multiple states?**
    ```python
    def setup_workflow():
        workflow = {
            "name": "Custom Process",
            "document_type": "Custom DocType",
            "states": [
                {
                    "state": "Draft",
                    "style": "Primary",
                    "doc_status": 0
                },
                {
                    "state": "Pending Approval",
                    "style": "Warning",
                    "doc_status": 0
                },
                {
                    "state": "Approved",
                    "style": "Success",
                    "doc_status": 1
                }
            ],
            "transitions": [
                {
                    "state": "Draft",
                    "action": "Submit for Approval",
                    "next_state": "Pending Approval",
                    "allowed": "Custom User"
                },
                {
                    "state": "Pending Approval",
                    "action": "Approve",
                    "next_state": "Approved",
                    "allowed": "Custom Approver"
                }
            ]
        }
        
        if not frappe.db.exists("Workflow", workflow["name"]):
            doc = frappe.get_doc({"doctype": "Workflow", **workflow})
            doc.insert()
    ```

32. **Q: How do you implement scheduled tasks?**
    ```python
    # hooks.py
    scheduler_events = {
        "daily": [
            "custom_app.tasks.daily_cleanup"
        ],
        "hourly": [
            "custom_app.tasks.send_reminders"
        ],
        "weekly": [
            "custom_app.tasks.weekly_report"
        ]
    }
    
    # tasks.py
    def daily_cleanup():
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    
    def send_reminders():
        # Send reminder emails
        pass
    ```

33. **Q: How do you implement document event handlers?**
    ```python
    # hooks.py
    doc_events = {
        "Sales Order": {
            "on_submit": "custom_app.events.so_on_submit",
            "on_cancel": "custom_app.events.so_on_cancel",
            "after_insert": "custom_app.events.so_after_insert"
        }
    }
    
    # events.py
    def so_on_submit(doc, method):
        # Create related documents
        create_project(doc)
        notify_departments(doc)
    
    def notify_departments(doc):
        frappe.publish_realtime(
            'sales_order_submitted',
            {"sales_order": doc.name}
        )
    ```

34. **Q: How do you implement custom notifications?**
    ```python
    def setup_notification():
        notification = frappe.new_doc("Notification")
        notification.name = "Custom Alert"
        notification.subject = "New {{ doc.doctype }} {{ doc.name }}"
        notification.document_type = "Custom DocType"
        notification.event = "Submit"
        notification.channel = "Email"
        notification.message = """
            Dear {{ doc.customer_name }},
            Your {{ doc.doctype }} {{ doc.name }} has been submitted.
        """
        notification.insert()
    ```

35. **Q: How do you implement email alerts?**
    ```python
    def send_custom_email(doc):
        frappe.sendmail(
            recipients=[doc.email],
            subject=f"Document {doc.name} Status Update",
            template="custom_email",
            args={
                "name": doc.name,
                "status": doc.status,
                "comments": doc.comments
            },
            header=["Status Update", "green"]
        )
    
    # custom_email.html
    <h3>Status Update</h3>
    <p>Document {{ name }} status has been updated to {{ status }}</p>
    {% if comments %}
    <p>Comments: {{ comments }}</p>
    {% endif %}
    ```

## Security and Permissions

36. **Q: How do you implement role-based access control?**
    ```python
    def setup_roles_and_permissions():
        # Create custom role
        if not frappe.db.exists("Role", "Custom Manager"):
            role = frappe.new_doc("Role")
            role.role_name = "Custom Manager"
            role.desk_access = 1
            role.insert()
        
        # Set permissions
        custom_manager_perms = {
            "read": 1,
            "write": 1,
            "create": 1,
            "delete": 1,
            "submit": 1,
            "cancel": 1,
            "amend": 1
        }
        
        add_permission(
            "Custom DocType",
            "Custom Manager",
            custom_manager_perms
        )
    ```

37. **Q: How do you implement custom authentication?**
    ```python
    from frappe.auth import LoginManager
    
    class CustomLoginManager(LoginManager):
        def validate_user(self):
            # Custom validation logic
            super().validate_user()
            
            if not self.user_in_approved_list():
                frappe.throw("User not approved")
        
        def user_in_approved_list(self):
            return frappe.db.exists(
                "Approved Users",
                {"user": self.user}
            )
    ```

38. **Q: How do you implement field-level permissions?**
    ```python
    def setup_field_permissions():
        docperm = frappe.new_doc("Custom DocPerm")
        docperm.parent = "Custom DocType"
        docperm.role = "Custom Role"
        docperm.permlevel = 1
        docperm.read = 1
        docperm.write = 0
        
        # Field permission in DocType
        custom_field = {
            "fieldname": "sensitive_data",
            "permlevel": 1
        }
        
        update_field_permission(
            "Custom DocType",
            custom_field
        )
    ```

39. **Q: How do you implement API authentication?**
    ```python
    @frappe.whitelist()
    def custom_api():
        # Check if user has permission
        if not frappe.has_permission("Custom DocType"):
            frappe.throw("Not permitted")
        
        # API token validation
        api_token = frappe.get_request_header("Authorization")
        if not validate_api_token(api_token):
            frappe.throw("Invalid API token")
        
        return "API response"
    
    def validate_api_token(token):
        return frappe.db.exists(
            "API Token",
            {"token": token, "enabled": 1}
        )
    ```

40. **Q: How do you implement user permissions?**
    ```python
    def setup_user_permissions():
        # Add user permission
        user_permission = frappe.new_doc("User Permission")
        user_permission.user = "user@example.com"
        user_permission.allow = "Custom DocType"
        user_permission.for_value = "CUST-001"
        user_permission.apply_to_all_doctypes = 1
        user_permission.insert()
        
        # Check user permission
        def validate_user_permission(doc, method):
            if not frappe.has_permission(
                doc.doctype,
                "write",
                doc
            ):
                frappe.throw("Not permitted")
    ```

## Deployment and Performance

41. **Q: How do you implement custom deployment configurations?**
    ```python
    # config.py
    from frappe.utils import cstr, cint
    
    def get_site_config():
        return {
            'maintenance_mode': 0,
            'pause_scheduler': 0,
            'developer_mode': 0,
            'disable_website_cache': 0,
            'rate_limit': {
                'limit': 100,
                'window': 3600
            }
        }
    ```

42. **Q: How do you implement caching strategies?**
    ```python
    from frappe.utils.caching import redis_cache
    
    @redis_cache
    def get_cached_data(key):
        # Expensive operation
        result = frappe.db.sql("""
            SELECT * FROM `tabCustom DocType`
            WHERE status = 'Active'
        """, as_dict=True)
        return result
    
    def clear_cache():
        frappe.cache().delete_key('cached_data')
    ```

43. **Q: How do you implement database optimization?**
    ```python
    def optimize_database():
        # Add indexes
        frappe.db.sql("""
            CREATE INDEX IF NOT EXISTS idx_status
            ON `tabCustom DocType` (status)
        """)
        
        # Optimize tables
        frappe.db.sql("""
            OPTIMIZE TABLE `tabCustom DocType`
        """)
        
        # Clear unused data
        frappe.db.sql("""
            DELETE FROM `tabError Log`
            WHERE creation < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
    ```

44. **Q: How do you implement load balancing?**
    ```python
    # nginx configuration
    def get_nginx_config():
        return """
        upstream frappe {
            server 127.0.0.1:8000 weight=1;
            server 127.0.0.1:8001 weight=1;
            server 127.0.0.1:8002 weight=1;
        }
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://frappe;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
        """
    ```

45. **Q: How do you implement monitoring and logging?**
    ```python
    import logging
    
    def setup_logging():
        # Custom logger
        logger = logging.getLogger("custom_app")
        logger.setLevel(logging.DEBUG)
        
        # File handler
        fh = logging.FileHandler("custom_app.log")
        fh.setLevel(logging.DEBUG)
        
        # Formatter
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        fh.setFormatter(formatter)
        logger.addHandler(fh)
        
        return logger
    
    # Usage
    logger = setup_logging()
    logger.info("Custom operation completed")
    ```

## Integration and Development

46. **Q: How do you implement external API integration?**
    ```python
    import requests
    
    class ExternalAPIIntegration:
        def __init__(self):
            self.api_key = frappe.conf.get("external_api_key")
            self.base_url = "https://api.external.com/v1"
        
        def make_request(self, endpoint, method="GET", data=None):
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            response = requests.request(
                method,
                f"{self.base_url}/{endpoint}",
                headers=headers,
                json=data
            )
            
            response.raise_for_status()
            return response.json()
    ```

47. **Q: How do you implement webhooks?**
    ```python
    @frappe.whitelist(allow_guest=True)
    def webhook_handler():
        if not validate_webhook_signature():
            frappe.throw("Invalid webhook signature")
        
        data = frappe.request.get_json()
        
        # Process webhook data
        process_webhook_data(data)
        
        return "Success"
    
    def validate_webhook_signature():
        signature = frappe.get_request_header("X-Webhook-Signature")
        # Validate signature
        return True
    ```

48. **Q: How do you implement custom REST endpoints?**
    ```python
    @frappe.whitelist()
    def custom_endpoint():
        try:
            # Get request parameters
            params = frappe.form_dict
            
            # Process request
            result = process_request(params)
            
            # Return response
            return {
                "status": "success",
                "data": result
            }
        except Exception as e:
            return {
                "status": "error",
                "message": str(e)
            }
    ```

49. **Q: How do you implement file handling?**
    ```python
    def handle_file_upload():
        file_doc = frappe.get_doc({
            "doctype": "File",
            "attached_to_doctype": "Custom DocType",
            "attached_to_name": "DOC-001",
            "attached_to_field": "attachment",
            "file_url": "",
            "file_name": "document.pdf",
            "is_private": 1,
            "content": frappe.request.files
