The goal of this project is to learn how to set up and manage an EntraID environment
Specifically managing users/groups, managing authentication, and setting up department-specific environments (SharePoint)
I am using the Microsoft Dev Program for this

There will be 5 departments: HR, Sales & Marketing, IT, Finance, Operations

--Groups--
  Each department will get its own group with the following settings
    Group type: Security -- This is for IAM purposes
    Group name: (department name)
    Microsoft Entra roles can be assigned to the group: No, this will be restricted to the administrator group
    Membership type: Dynamic User, with the query (user.department -contains "(department name)")
