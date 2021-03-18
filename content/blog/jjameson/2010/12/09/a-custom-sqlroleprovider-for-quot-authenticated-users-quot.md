---
title: "A Custom SqlRoleProvider for \"Authenticated Users\""
date: 2010-12-09T07:25:00-07:00
excerpt:
  "Prior to the recent \"v2\" release on my current project, we had been using
  the ASP.NET SqlRoleProvider to manage the various roles used by the Web site.
  Over a month ago, someone contacted me about an issue he was encountering with
  a specific user...."
aliases:
  [
    "/blog/jjameson/archive/2010/12/08/a-custom-sqlroleprovider-for-quot-authenticated-users-quot.aspx",
    "/blog/jjameson/archive/2010/12/09/a-custom-sqlroleprovider-for-quot-authenticated-users-quot.aspx",
  ]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Web Development", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/12/09/a-custom-sqlroleprovider-for-quot-authenticated-users-quot.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/09/a-custom-sqlroleprovider-for-quot-authenticated-users-quot.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/09/a-custom-sqlroleprovider-for-quot-authenticated-users-quot.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Prior to the recent "v2" release on my current project, we had been using the
ASP.NET
**[SqlRoleProvider](http://msdn.microsoft.com/en-us/library/system.web.security.sqlroleprovider.aspx)**
to manage the various roles used by the Web site.

Over a month ago, someone contacted me about an issue he was encountering with a
specific user. The problem turned out to be caused by the fact that the user was
not added to the **Authenticated Users** role.

If you've ever used Forms-Based Authentication (FBA) against Active Directory,
then you've likely used built-in groups like **NT AUTHORITY\Authenticated
Users** to restrict access to specific content. Unfortunately, if you want to do
the same thing when using FBA with users and roles stored in SQL Server, you
have to explicitly create the **Authenticated Users** role and ensure that every
user is added to the role.

Shortly after we identified the root cause of the issue I mentioned earlier, I
created a work item in Team Foundation Server to come up with a solution for
avoiding this issue in the future:

> **Title:** Membership in the "Authenticated Users" role should be implicit
>
> **Description:
> ** Administrators shouldn't have to explicitly add users to the "Authenticated
> Users" role. Instead, membership in this role should be implicit, since anyone
> that specifies a username/password is considered "authenticated".
>
> Unfortunately, the out-of-the-box role provider (SqlRoleProvider) that we are
> currently using doesn't function this way.

In the "v2" solution, I created a custom role provider that automatically
manages the **Authenticated Users** role (in other words, administrators do not
need to create the role in the underlying database or explicitly add users to
the role).

Fortunately, the ASP.NET **SqlRoleProvider** is not marked as `sealed` and
therefore it requires very little code to implement the custom role provider.

I started by overriding the **RoleExists** method:

```
        public override bool RoleExists(
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                return true;
            }

            return base.RoleExists(roleName);
        }
```

As you can see, this simply checks if the "Authenticated Users" role was
specified, in which case it returns `true` regardless of whether the role exists
in the underlying database.

Next, I proceeded to override the **IsUserInRole** method:

```
        public override bool IsUserInRole(
            string username,
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                return true; // assume the specified user is valid
            }

            return base.IsUserInRole(username, roleName);
        }
```

Notice the comment that I added to the `return` statement. At first, I thought
about validating the specified user (to ensure the user actually exists), but
then I thought "Why take the extra database hit (i.e. calling the
**Membership.GetUser** method) just to ensure the user exists?"

The reality is that the **IsUserInRole** method will never get called with an
invalid user (since the user wouldn't be able to login in the first place).

I then moved on to something slightly more challenging - implementing the
**GetRolesForUser** method:

```
        public override string[] GetRolesForUser(
            string username)
        {
            string[] sqlRoles = base.GetRolesForUser(username);

            string[] roles = AddAuthenticatedUsersRole(sqlRoles);

            return roles;
        }
```

As you can see, I'm simply augmenting the list of roles specified for the user
in the database with the additional **Authenticated Users** role.

As I mentioned before, we were previously using the **SqlRoleProvider** and thus
the **Authenticated Users** role had already been added to the database. Even
though I added a section to the v2.0 installation guide to explicitly delete the
**Authenticated Users** role prior to upgrading, I still thought it wise to
avoid "blindly" adding the **Authenticated Users** role to the list of roles
returned from the database (potentially adding a duplicate). Consequently, in
the helper method I check to see if the role is already specified before
prepending it to the list:

```
        private static string[] AddAuthenticatedUsersRole(
            string[] sqlRoles)
        {
            Debug.Assert(sqlRoles != null);

            foreach (string roleName in sqlRoles)
            {
                if (string.Compare(
                    roleName,
                    authenticatedUsersRoleName,
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    // The "Authenticated Users" role hasn't been removed from
                    // the database
                    return sqlRoles;
                }
            }

            string[] roles = new string[sqlRoles.Length + 1];
            roles[0] = authenticatedUsersRoleName;

            sqlRoles.CopyTo(roles, 1);
            return roles;
        }
```

After overriding a few more methods (e.g. **DeleteRole**), I then swapped out
the **SqlRoleProvider** with the custom role provider. That's when I encountered
the following error in IIS Manager (after clicking **.NET Roles**):

> This feature cannot be used because the default provider is not a trusted
> provider.
>
> You can use this feature only when the default provider is a trusted provider.
> If you are a server administrator, you can make a provider a trusted provider
> by adding the provider type to the trusted providers list in the
> Administration.config file. The provider has to be strongly typed and added to
> the GAC (Global Assembly Cache).

I then added the following steps to the installation guide:

> To add the custom role provider to the IIS Administration.config file:
>
> 1. Click **Start**, point to **All Programs**, point to **Accessories**, and
>    right-click **Command Prompt**, and then click **Run as administrator**.
>
> 2. At the command prompt, change to the following directory:\
>    \
>    **%WinDir%\system32\inetsrv\config
>    **
>
> 3. Type the following command:
>    
>    ```
>    notepad administration.config
>    ```
>
> 4. In the /configuration/system.webServer/management/trustedProviders section, add the following:
>    
>    ```
>    <add
>      type="Fabrikam.Portal.Web.Security.FabrikamSqlRoleProvider,
>        Fabrikam.Portal.Web, Version=2.0.0.0, Culture=neutral,
>        PublicKeyToken=c8cdcbca6f69701f" />
>    ```
>
> 5. Save the changes to the file and close Notepad.

Once I had made the custom role provider a trusted provider, I was able to
verify some basic functionality and fix a couple of bugs. For example, I
discovered that while it's okay to throw an exception in the **FindUsersInRole**
method when the "Authenticated Users" role is specified, IIS Manager doesn't
like it when you throw an exception from the **GetUsersInRole** method. If you
throw an exception in the **GetUsersInRole** method, then it prohibits the list
of roles from being displayed in IIS Manager. Consequently, I simply chose to
return an empty list of users when **GetUsersInRole** is called for the
"Authenticated Users" role.

An alternative that I briefly considered for the **GetUsersInRole** method was
to simply query the database for all users, but that just didn't feel right in
my gut. It's probably not an issue when you have at most a few thousand users,
but I was concerned about performance with tens of thousands of users.

Here is the complete source for the custom role provider:

```
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Globalization;
using System.Web.Security;
using Fabrikam.Portal.CoreServices.Logging;

namespace Fabrikam.Portal.Web.Security
{
    public class FabrikamSqlRoleProvider : SqlRoleProvider
    {
        private const string authenticatedUsersRoleName =
            "Authenticated Users";

        /// <summary>
        /// Adds the "Authenticated Users" role to the list of roles retrieved
        /// from the database.
        /// </summary>
        private static string[] AddAuthenticatedUsersRole(
            string[] sqlRoles)
        {
            Debug.Assert(sqlRoles != null);

            foreach (string roleName in sqlRoles)
            {
                if (string.Compare(
                    roleName,
                    authenticatedUsersRoleName,
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    // The "Authenticated Users" role hasn't been removed from
                    // the database
                    return sqlRoles;
                }
            }

            string[] roles = new string[sqlRoles.Length + 1];
            roles[0] = authenticatedUsersRoleName;

            sqlRoles.CopyTo(roles, 1);
            return roles;
        }

        /// <summary>
        /// Adds the specified user names to each of the specified roles.
        /// </summary>
        /// <param name="usernames">A string array of user names to be added to
        /// the specified roles.</param>
        /// <param name="roleNames">A string array of role names to add the
        /// specified user names to.</param>
        public override void AddUsersToRoles(
            string[] usernames,
            string[] roleNames)
        {
            if (roleNames == null)
            {
                throw new ArgumentNullException("roleNames");
            }
            else if (roleNames.Length == 0)
            {
                // Let the .NET Framework handle the error
                // (i.e. "The array parameter 'roleNames' should not be empty.")
                base.AddUsersToRoles(usernames, roleNames);
                return;
            }

            List<string> filteredRoleList = new List<string>();

            foreach (string roleName in roleNames)
            {
                if (string.Compare(
                    roleName,
                    authenticatedUsersRoleName,
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    Logger.LogWarning(
                        CultureInfo.CurrentCulture,
                        "Users cannot be explicitly added to the '{0}' role.",
                        authenticatedUsersRoleName);
                }
                else
                {
                    filteredRoleList.Add(roleName);
                }
            }

            if (filteredRoleList.Count == 0)
            {
                return;
            }

            string[] filteredRoles = filteredRoleList.ToArray();

            base.AddUsersToRoles(usernames, filteredRoles);
        }

        /// <summary>
        /// Adds a new role to the role database.
        /// </summary>
        /// <param name="roleName">The name of the role to create.</param>
        public override void CreateRole(
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                string message = string.Format(
                   CultureInfo.CurrentCulture,
                   "The '{0}' role already exists.",
                   roleName);

                throw new ArgumentException(
                    message,
                    "roleName");
            }

            base.CreateRole(roleName);
        }

        /// <summary>
        /// Removes a role from the role database.
        /// </summary>
        /// <param name="roleName">The name of the role to delete.</param>
        /// <param name="throwOnPopulatedRole">If <c>true</c>, throws an exception
        /// if roleName has one or more members.</param>
        /// <returns><c>true</c> if the role was successfully deleted;
        /// otherwise, <c>false</c>.</returns>
        public override bool DeleteRole(
            string roleName,
            bool throwOnPopulatedRole)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                // Allow the "Authenticated Users" role to be deleted
                // if and only if it exists in the database (i.e. the
                // role created previously with the ASP.NET SqlRoleProvider
                // was not properly cleaned up prior to switching over to
                // the custom role provider.
                if (base.RoleExists(roleName) == false)
                {
                    string message = string.Format(
                        CultureInfo.CurrentCulture,
                        "The '{0}' role cannot be deleted.",
                        authenticatedUsersRoleName);

                    throw new ArgumentException(
                        message,
                        "roleName");

                }
            }

            return base.DeleteRole(roleName, throwOnPopulatedRole);
        }

        /// <summary>
        /// Gets an array of user names in a role where the user name contains
        /// the specified user name to match.
        /// </summary>
        /// <param name="roleName">The role to search in.</param>
        /// <param name="usernameToMatch">The user name to search for.</param>
        /// <returns>A string array containing the names of all the users where
        /// the user name matches usernameToMatch and the user is a member of
        /// the specified role.</returns>
        public override string[] FindUsersInRole(
            string roleName,
            string usernameToMatch)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                string message = string.Format(
                    CultureInfo.CurrentCulture,
                    "This method is not supported for the '{0}' role.",
                    authenticatedUsersRoleName);

                throw new ArgumentException(message);
            }

            return base.FindUsersInRole(roleName, usernameToMatch);
        }

        /// <summary>
        /// Gets a list of all the roles for the application.
        /// </summary>
        /// <returns>A string array containing the names of all the roles stored
        /// in the database for a particular application.</returns>
        public override string[] GetAllRoles()
        {
            string[] sqlRoles = base.GetAllRoles();

            string[] roles = AddAuthenticatedUsersRole(sqlRoles);

            return roles;
        }

        /// <summary>
        /// Gets a list of the roles that a user is in.
        /// </summary>
        /// <param name="username">The user to return a list of roles for.
        /// </param>
        /// <returns>A string array containing the names of all the roles that
        /// the specified user is in.</returns>
        public override string[] GetRolesForUser(
            string username)
        {
            string[] sqlRoles = base.GetRolesForUser(username);

            string[] roles = AddAuthenticatedUsersRole(sqlRoles);

            return roles;
        }

        /// <summary>
        /// Gets a list of users in the specified role.
        /// </summary>
        /// <param name="roleName">The name of the role to get the list of users
        /// for.</param>
        /// <returns>A string array containing the names of all the users who
        /// are members of the specified role.</returns>
        public override string[] GetUsersInRole(
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                // HACK: If we throw an exception here, it prohibits the list of
                // roles from being displayed in IIS Manager. Therefore just
                // return an empty list.
                return new string[0];
            }

            return base.GetUsersInRole(roleName);
        }

        /// <summary>
        /// Gets a value indicating whether the specified user is in the
        /// specified role.
        /// </summary>
        /// <param name="username">The user name to search for.</param>
        /// <param name="roleName">The role to search in.</param>
        /// <returns><c>true</c> if the specified user name is in the
        /// specified role; otherwise, <c>false</c>.</returns>
        public override bool IsUserInRole(
            string username,
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                return true; // assume the specified user is valid
            }

            return base.IsUserInRole(username, roleName);
        }

        /// <summary>
        /// Removes the specified user names from the specified roles.
        /// </summary>
        /// <param name="usernames">A string array of user names to be removed
        /// from the specified roles.</param>
        /// <param name="roleNames">A string array of role names to remove the
        /// specified user names from.</param>
        public override void RemoveUsersFromRoles(
            string[] usernames,
            string[] roleNames)
        {
            if (roleNames == null)
            {
                throw new ArgumentNullException("roleNames");
            }
            else if (roleNames.Length == 0)
            {
                // Let the .NET Framework handle the error
                // (i.e. "The array parameter 'roleNames' should not be empty.")
                base.AddUsersToRoles(usernames, roleNames);
                return;
            }

            List<string> filteredRoleList = new List<string>();

            foreach (string roleName in roleNames)
            {
                if (string.Compare(
                    roleName,
                    authenticatedUsersRoleName,
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    Logger.LogWarning(
                        CultureInfo.CurrentCulture,
                        "Users cannot be explicitly removed from the '{0}'"
                            + " role.",
                        authenticatedUsersRoleName);
                }
                else
                {
                    filteredRoleList.Add(roleName);
                }
            }

            if (filteredRoleList.Count == 0)
            {
                return;
            }

            string[] filteredRoles = filteredRoleList.ToArray();

            base.RemoveUsersFromRoles(usernames, filteredRoles);
        }

        /// <summary>
        /// Gets a value indicating whether the specified role name already
        /// exists in the role database.
        /// </summary>
        /// <param name="roleName">The name of the role to search for in the
        /// database.</param>
        /// <returns><c>true</c> if the role name already exists in the
        /// database; otherwise, <c>false</c>.</returns>
        public override bool RoleExists(
            string roleName)
        {
            if (string.Compare(
                roleName,
                authenticatedUsersRoleName,
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                return true;
            }

            return base.RoleExists(roleName);
        }
    }
}
```

Note that there are a few issues in IIS Manager when using this custom role
provider:

- As described earlier, the **GetUsersInRole** method always returns an empty
  list for the "Authenticated Users" role. Consequently, IIS Manager will always
  display "0" for the number of users in the role.
- Immediately after creating a new user, if you view the role membership for the
  user, then IIS Manager does not show **Authenticated Users** role as checked.
  This appears to be a caching issue in IIS Manager. Even if you don't check the
  Authenticated Users role when creating the user, the next time
  `IsUserInRole(username, "Authenticated Users")` is called, it will definitely
  return `true` -- and, similarly, the **GetRolesForUser** method will include
  "Authenticated Users" in the returned list.

Somehow I doubt that I'll ever use the ASP.NET **SqlRoleProvider** again ;-)
