# The Issue
Say you mis-typed a password when creating an account in MySQL. Or perhaps, like me, you ended up with a word in your randomly generated password that you didn't like (for example, you got "imprudent" as one of your words, but you always confuse it for the similarly-spelled "impudent" and that 'r' really trips you up).

Regardless, you don't like your password, and you want to change it. This process is actually pretty simple, but you may run into a few roadblocks, which we'll discuss as they pop up.

# The Process
First, we'll get into MySQL using root. To do so, use the following commands: 
```
sudo mysql -u root
use mysql;
```

Now that we're in MySQL, we can pull up a list of users with the command `select user from user;`. The result will look similar to this:\
<img width="1280" height="983" alt="image" src="https://github.com/user-attachments/assets/7dffe422-8199-4524-83c7-adcd652bf4be" />


It's also a good idea to know for certain to which host the user belongs, so you may want to instead use `select user, host from user;`. The resulting table will look like this:\
<img width="1280" height="983" alt="image" src="https://github.com/user-attachments/assets/f69429eb-2a28-4a8c-94fb-13f9a7cb2b0d" />


For the purpose of this documentation, I'll be altering the user `demo_user`, the password for which is currently set as `ImprudentLobster`.

To change `demo_user`'s password so it doesn't have that pesky 'r' anymore, simply type `ALTER USER demo_user@localhost IDENTIFIED BY 'ImpudentLobster';`
<img width="1280" height="983" alt="image" src="https://github.com/user-attachments/assets/fa838d46-d999-49c9-838e-225985e19c68" />

You should also `FLUSH PRIVILEGES;` after changing a password, just to be on the safe side. This will ensure that MySQL reloads the user table with the new information.

# Some Problems
Since we used pretty lax password security for this project, you may end up in a situation where you get an error similar to this:`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`

The likely cause of this is your database's password validation policy. We used `LOW` for this project, but for some reason mine kept switching itself back to `MEDIUM`. You can check the current value by using the command `SHOW variables LIKE 'validate_password%';`:
<img width="1280" height="984" alt="image" src="https://github.com/user-attachments/assets/04057872-9398-4223-82ca-d191875a0a8d" />

If your table, like mine, shows `validate_password.policy` as `MEDIUM`, you can change this setting with the command `SET GLOBAL validate_password.policy=LOW;`
<img width="1280" height="984" alt="image" src="https://github.com/user-attachments/assets/7da36660-fbe7-4208-afaf-c0b5c05ad79c" />

Now you should be able to change your password without getting the error. Alternatively, just come up with an entirely new password that satisfies the higher security policy.

# Bad Advice
Now we come to the "Anarchist's Cookbook" section of the documentation, where I tell you about something I 100% should not have done, but did anyway because I'm an adult and I do what I want.

When this happened to me for real, I kept getting `ERROR 1819 (HY000)`, even when my `validate_password.policy` was set to `LOW`. I couldn't figure out what the issue was, so I decided to just start breaking stuff until I got what I wanted. It's a strategy that has about a 50% success rate, which is good enough for my impulsive brain.

With that said, here's how you circumvent the password policy by ripping out its still-beating heart, changing your password with no constraints, then shoving it back in.

$\color{Red}\large{\textbf{I WOULD NOT RECOMMEND DOING THIS ON A PRODUCTION SERVER.}}$\
$\color{Red}\large{\textbf{I ONLY DID THIS BECAUSE I DON'T CARE WHAT HAPPENS WITH THIS VM.}}$

The code for uninstalling password validation is `UNINSTALL COMPONENT 'file://component_validate_password';`. Once you do this, you can change user passwords to whatever you want because there are no rules about password length, etc. Once you've done this, IMMEDIATELY reinstall by using `INSTALL COMPONENT 'file://component_validate_password';`. I really have no idea why I had to do this in the first place, as I can't recreate the issue today. But there it is.

# Removing a User Entirely
Just for fun, let's remove `demo_user`, since I won't be using them anymore. You could consider this another workaround for the password issue (just remove them and create them again), assuming there aren't any important privileges associated with the user.

The command to remove a user is `DROP`, as in `DROP USER 'demo_user'@'localhost';`
<img width="1280" height="984" alt="image" src="https://github.com/user-attachments/assets/899cfa29-35e4-4b58-bfce-4ee3922250d7" />
