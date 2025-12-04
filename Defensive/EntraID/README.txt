The goal of this project is to learn how to set up and manage an EntraID environment
Specifically managing users/groups, managing authentication, and setting up department-specific environments (SharePoint)
I am using the Microsoft Dev Program for this

There will be 5 departments: HR, Sales & Marketing, IT, Finance, Operations

--Groups--
  Each department will get its own group with the following settings
    Group type: Security -- This is for IAM purposes
    Group name: (department name)
    Microsoft Entra roles can be assigned to the group: No, this will be restricted to the administrator group
    Membership type: Dynamic User, with the query 
      user.department -contains "(department name)"

  There will be an additional group for Managers, with the same format as above, except the membership queries will be like below
    (user.jobTitle -contains "Manager")
	  (user.jobTitle -eq "President")

  The last group will be administrators, which will be manually assigned.

--Setup Users--
	I already have the default 16 randomly generated users, and myself as the 17th user
	I will put 3 users in each department, with generic titles
	1 person in each department is going to be a manager
	The last user will be the president in no significant department, and he will just be assigned as the manager for every manager
	I will be the 4th member of the IT department with the IAM Engineer role

--Authentication
	First, I will review MFA. The default settings are fairly safe, but we can change a few things.
	I will set up 2 different authentication strengths
		Low Risk: Password + Authentication App or Passkey or TAP -- For low-risk accounts or low-risk activities
		High Risk: Passkey Only                                   -- For high-risk Activities

  This setup is not the most user-friendly, but it is very secure and is slowly becoming the standard. 
  In a real environment, you'd likely want a backup authentication that is heavily monitored but less strict in case global administrators lose access.

--Sign In Policies
	I will create the first policy named Baseline
	It targets all users, all resources, and it will grant access based on the authentication strength, Low Risk
	
	I will create the second policy named Admin
	It targets the admin group, all resources, and it will grant access based on the authentication strength, High Risk

--Group Specific Sharepoints
	For each department
  	Create SharePoint -> Team site -> Standard Team -> Name/email/address based on department
  	Go to SharePoint website -> Gear icon in top right -> Site Permissions -> Advanced Permission Settings
  	Click (department name) Members > New > Invite people > Then type the group name
