# Sign in

1. To begin, log into the virtual machine using the following credentials: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++

2. Open **Microsoft Edge**, go to `https://github.com/enterprises/skillable-events/sso`, and sign in with the username and Temporary Access Pass (TAP). Use the password only if you're asked for one instead. Keep this page open.

    - **Username:** +++@lab.CloudPortalCredential(User1).Username+++
    - **Password:** +++@lab.CloudPortalCredential(User1).Password+++
    - **TAP:** +++@lab.CloudPortalCredential(User1).AccessToken+++

3. Open **Visual Studio Code**, select **Sign in**, then **Continue with GitHub**. Choose the single sign-on option when it appears and complete authentication through the SSO page you left open.

!IMAGE[sp533k2e.png](instructions361694/sp533k2e.png)

> [+Hint] GitHub sign-in fails
>
> Wait a minute and retry. Your lab account is provisioned to GitHub after the lab starts.
