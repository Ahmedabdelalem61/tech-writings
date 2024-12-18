# Odoo Access Rights in a Nutshell

> **Objective:**
> Build a simple  understanding of Odoo security in a nutshell, including access rights, record rules, and group logic. Explore hierarchical steps for mastering deeper security concepts.

---

## 📦 Description

This section provides a comprehensive understanding of Odoo's access control mechanisms, including **access rights**, **record rules**, and how they interact. The structure builds knowledge step-by-step to explore Odoo security concepts in depth.

---

## 🛠 Simple ACL (Access Control List)

**General Overview:**
Access Control Lists (ACLs) define basic permissions like `read`, `write`, `create`, and `unlink` for specific user groups.

---

## 📜 Record Rules

### **General Overview**

Record rules allow more granular control over what records users can see or modify. These rules are applied dynamically at the model level and provide flexibility to tailor access based on specific conditions.

---

### **Overriding Record Rules**

Overriding record rules involves modifying or adding rules to meet specific business needs. When doing so:

- Ensure that custom rules don’t unintentionally open security gaps.
- Always test thoroughly, especially in multi-user environments, to avoid unexpected behavior.

---

### **Scenarios for Overriding Record Rules**

#### **Scenario 1: Restrict Access to Records by Department**

**Default Rule:**
By default, users in the group `group_hr_user` can only access employee records who have a department same as their own department. The rule might look like this:

~~~
<record id="ir_rule_employee_own_department" model="ir.rule">
    <field name="name">Access Own Department</field>
    <field name="model_id" ref="hr.model_hr_employee"/>
    <field name="domain_force">[('department_id', '=', user.department_id.id)]</field>
    <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
</record>
~~~

**Override Rule to Expand Access:**
You want HR managers(`group_hr_manager` group) to access records for **all employees**, not just their department. Here’s the override:

~~~
<record id="ir_rule_employee_all_departments" model="ir.rule">
    <field name="name">Access All Departments</field>
    <field name="model_id" ref="hr.model_hr_employee"/>
    <field name="domain_force">[(1, '=', 1)]</field> <!-- Unrestricted access -->
    <field name="groups" eval="[(4, ref('hr.group_hr_manager'))]"/>
</record>
~~~

`[(1, '=', 1)]` is a special domain which allows access to all records (because the statement always evaluates to `true`).

---

#### **Scenario 2: Restrict Read Access to Sensitive Records**

**Default Rule:**
All users in the group `group_project_user` can see all project tasks, regardless of who created them. This is the default behavior:

~~~
<record id="ir_rule_project_task_default" model="ir.rule">
    <field name="name">Access All Tasks</field>
    <field name="model_id" ref="project.model_project_task"/>
    <field name="domain_force">[(1, '=', 1)]</field>
    <field name="groups" eval="[(4, ref('project.group_project_user'))]"/>
</record>
~~~

**Override Rule to Restrict Access:**
You override the rule to limit access to only the tasks assigned to the user:

~~~
<record id="ir_rule_project_task_assigned_only" model="ir.rule">
    <field name="name">Access Assigned Tasks Only</field>
    <field name="model_id" ref="project.model_project_task"/>
    <field name="domain_force">[('user_id', '=', user.id)]</field>
    <field name="groups" eval="[(4, ref('project.group_project_user'))]"/>
</record>
~~~

#### **Scenario 3: Override Existing Rule to Restrict Access**

#### **Original Rule (Using `department_id`)**

This rule restricts access to employees based on the user's department. It is defined as follows:

~~~
<record id="module_name.rule_department_access" model="ir.rule">
    <field name="name">Access by Department</field>
    <field name="model_id" ref="hr.model_hr_employee"/>
    <field name="domain_force">[('department_id', 'in', user.department_ids.ids)]</field>
    <field name="groups" eval="[(4, ref('module_name.group_a'))]"/>
</record>
~~~

#### **Override Rule (Using `branch_id`)**

We override the rule by using the same `id` (`module_name.rule_department_access`), but change the `domain_force` field to enforce access based on `branch_id` instead of `department_id`:

~~~
<record id="module_name.rule_department_access" model="ir.rule">
    <field name="name">Access by Branch</field>
    <field name="domain_force">[('branch_id', 'in', user.department_ids.ids)]</field>
</record>
~~~

Note that other fields we did not override (such as `model_id` or `groups`) are left with the same values they had in the original record rule.

## 👥 Groups

**General Overview:**
Groups define collections of users with shared access rights. Groups often imply other groups for hierarchical permissions. Implication is done using the field `implied_ids`.

### **Implication Logic**
- Group A implies Group B means that all the permissions of Group B are automatically granted to Group A.

~~~
<record id="group_user" model="res.groups">
    <field name="name">User</field>
</record>

<record id="group_manager" model="res.groups">
    <field name="name">Manager</field>
    <field name="implied_ids" eval="[(4, ref('module_name.group_user'))]"/>
</record>
~~~

Here, all the permissions of the User group are automatically granted to the Manager group.

### **When to Use [OR - AND] in Groups**
- **OR**: Combine permissions when any condition can suffice.
- **AND**: Restrict permissions by requiring all conditions to be met.

---

## ❓ Questions

1. **Does visibility depend on the group only or the rules applied to the group?**
   - No, if a user has a group that matches the menu’s group, the menu is visible, regardless of the ACL or record rules.

2. **Does combining two rules without `perm_read` show any records?**
   - No, in this case, all records will be visible.

3. **What is the difference in the behaviour of stored and non-stored computed fields when using record rules?**
   - Computed fields can be given the parameter `compute_sudo` when defined, this parameter controls whether these fields should be computed using the sudo access (bypass access rights) or not.
   - **Stored Computed Fields**: Have `compute_sudo` set to `true` by default.
   - **Non-Stored Computed Fields**: Have `compute_sudo` set to `false` by default, which means its value is different depending on the currently logged in user!

---

## 🛡 The Power of `sudo`

Using `sudo()` method on the `self` (or any recordset) in Odoo bypasses access rights and record rules. It’s powerful for background operations but must be used cautiously to avoid introducing security vulnerabilities (e.g. it can expose sensitive data or allow unauthorized changes).

---

# 🏗 How to Build Maintainable System Access Rights?

- Start with **simple ACLs** and incrementally add complexity.
- Use groups to create modular permission sets.
- Combine record rules logically for layered security.
- Test permissions with both **admin** and **non-admin** accounts.

# ✏️**Corrections or Suggestions**

If you notice any mistakes or areas for improvement, please let me know as quickly as possible. Your feedback is appreciated! reach me at https://www.linkedin.com/in/numerician/
