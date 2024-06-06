---
title: "How to edit or create new records on popup tree view?"
date: 2024-05-27 10:00:00 +0000
author: alexandre
categories: [Odoo]
tags: [Odoo, view]
description: "How to edit or create new records on popup tree view?"
---

## How to edit or create new records on popup tree view?

### Conclusion

In Odoo 17, popup tree view is not original support for creating records.

### Introduction

When we search for records for a m2o field on the form view, click `Search More` it will popup a tree view for the related model. 

![mto_search_more.png](/images/How_to_edit_or_create_new_records_on_popup_tree_view/m2o_search_more.png)

There are buttons at the bottom allowing us to create records for that model.

![m2o_tree_view.png](/images/How_to_edit_or_create_new_records_on_popup_tree_view/m2o_tree_view.png)

Here is the scenario, I wanna create a action to return a popup tree view like that, how we achieve feature that?

Create a server action:
```python
def action_open_attachments(self):
    self.ensure_one()
    view_id_tree = self.env.ref('module.view_attachment_tree').id

    return {
        'type': 'ir.actions.act_window',
        'name': _('Attachments'),
        'view_mode': 'tree',
        'res_model': 'module.attachment',
        'target': 'new',  # make target to 'new' in order to popup
        'views': [(view_id_tree, 'tree')],
    }
```
It turns out that whatever we add for which keys like `domain`, `context` or add attributes for view tag like `edit`,`create`, there is no effect for create records.

![m2o_tree_view_no_button.png](/images/How_to_edit_or_create_new_records_on_popup_tree_view/m2o_tree_view_no_button.png)

### Solution

For my scenario, I got a model A, and model B(attachment), in our model A we have a related m2o field to model B, so i create a simple form view for model A, which only contains a tree view on it, and then we return that form view.

```xml
<record id="view_modelA_form_popup" model="ir.ui.view">
    <field name="name">modelA.form.popup</field>
    <field name="model">modelA</field>
    <field name="arch" type="xml">
        <form string="Equipment">
            <sheet>
                <field name="attachment_ids" >
                    <tree string="Upload files" editable="bottom">
                        <field name="name"/>
                        <field name="note"/>
                        <field name="attachment_title" column_invisible="1"/>
                        <field name="attachment" filename="attachment_title" widget="binary"/>
                    </tree>
                </field>
            </sheet>
        </form>
    </field>
</record>
```
Modify the server action to return this view:

```python
def action_open_attachments(self):
    self.ensure_one()
    view_id_form = self.env.ref('module.view_modelA_form_popup').id

    return {
        'type': 'ir.actions.act_window',
        'name': _('Attachments for %s', self.name),
        'view_mode': 'form',
        'res_model': 'modelA',
        'res_id': self.id,
        'target': 'new',  # make target to 'new' in order to popup
        'views': [(view_id_form, 'form')],
    }

```
Now we can edit and create records for m2o field on that tree view like form view.

![m20_form_solution.png](/images/How_to_edit_or_create_new_records_on_popup_tree_view/m20_form_solution.png)

---
We need have a other model to open that tree view in the form view or create a javascript extension.
