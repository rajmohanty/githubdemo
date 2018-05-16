AWS > SSM > Instance Patch Management
    **Enforce: Enabled with Target Patch Group**
    *Set 'turbot:PatchingMaintenanceWindow' and 'Patch Group' tags for an instance based on the value of 'AWS > EC2> Instance Patching Window' and AWS > EC2 Instance Target Patch Group*

AWS > EC2 > Instance Approved Patch Groups
    **YAML List**
    *A list patch group names that are approved for use.*

AWS > EC2 > Instance Target Patch Group
    **Text Value**
    *The patch group name that will be enforced as the Target patch group, or used as the initial patch group if enforcing with approved patch group. Must be part of the approved patch group list to be valid*

AWS > EC2 > Instance Patching Maintenance Window
    **Text Value**
    *Set the maintenance window to be used for automatic patching. To be valid the selected maintenance window must be defined in the SSM > Maintenance Window Definition*

AWS > EC2 > Instance Target Patch Group
    **Text Value**
    *The patch group name that will be enforced as the Target patch group, or used as the initial patch group if enforcing with approved patch group. Must be a part of the approved patch group list to be valid.*

AWS > EC2 > Inventory Collection for Bob's Demo Account
    **Enforce: Inventory collection enabled**
    *Apply the turbot:InventoryCollection tag to the instance and set the value to be true.*






