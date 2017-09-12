# Backport Request Automator

## Features

* Cherry pick commits based off of LPS number
* Create the BPR ticket
* Send the pull request
* Progress the BPR ticket through the workflow

## Prerequisites

#### Curl and .netrc

This script makes use of JIRA's REST API via curl. In order to streamline the process, we can save our credentials in a **.netrc** file. To do this, create a **.netrc** file within your home directory making it readible and writable only by the owner. The syntax for this file is as follows:

```
machine issues.liferay.com login XXXX password XXXX
```

where **login** is your JIRA screen name and **password** is your JIRA password.

#### Hub

Hub is a wrapper for Git that is tailed for GitHub. Hub makes it much simpler to send pull requests. To set up Hub please visit - https://hub.github.com/

#### Bash functions

Copy the functions from the [bpr](https://github.com/jonathanmccann/backport-automator/blob/master/bpr) file to your ```.bash_aliases``` file so that you are able to call the functions from the command line.

At the start of the ```bpr``` file there are four configurable values:

```CURRENTUSERGITHUBNAME```, ```CURRENTUSERJIRANAME```, ```DEFAULTGITHUBREVIEWER```, and ```DEFAULTJIRAREVIEWER```

For the current user values, enter in your GitHub and Jira usernames. For the reviewer values, enter in the GitHub and Jira usernames of the reviewer your normally send backport requests to.

Note that the reviewer values are simply defaults. If you wish to submit the backport to a different review, you will be able to specify their name on the fly.

## Usage

### Preparing to Backport

This function assumes that you are either backporting to **ee-6.2.x** or **7.0.x**. Other versions are not supported at this time.

To begin, be sure that your **EE master branch** is up to date and contains the fix that you wish to backport. Then checkout either **ee-6.2.x** or **7.0.x**. There is no need to create a new branch as the function does that for you.

### Beginning Backport

The basic command is ```bpr LPS-12345```. Running this will check out a new branch, cherry pick commits that have ```LPS-12345``` in their commit message, push the branch to your origin, create or utilize a previously created BPR ticket, send the pull request, and progress the BPR ticket through the proper workflow.

However, cherry picking can be tricky and some times there are conflicts. When that happens, the function will exit with one of the following messages:

```
Could not perform cherry pick for 3706ecc7c2f2 LPS-12345 Commit Message
Please resolve the conflict(s) and then run 'bpr -s LPS-12345'
```

or

```
Could not perform cherry pick for 3706ecc7c2f2 LPS-12345 Commit Message
Please resolve the conflict(s) and then run 'bpr -h LPS-12345 3706ecc7c2f2'
```

When a cherry picking error is occurred, you must manually resolve the conflict and then run the provided command. If there are more conflict errors then the above will repeat.

Once all of the commits have been successfully applied, then you are able to test your changes or submit the backport.

There are two distinct flags that are used and seen in the above messages:

##### Submit Flag

The ```-s``` flag signifies to the function that the current branch is done cherry picking and ready to be pushed through the BPR workflow. Note that the function will not ask if you wish to test the changes, so please do so before running this command.

It requires a single argument and the argument is the **LPS number**.

##### Hash Flag

The ```-h``` flag signifies to the function that there are more commits that need to be cherry picked. It will then begin cherry picking on the specified commit hash. If there are no more conflicts, it will then push the current branch through the BPR workflow.

It requires two arguments. The first argument is the **LPS number** and the second argument is the next **commit hash** that the function needs to cherry pick.

### After Cherry Picking Has Finished

Once the cherry picking has finished, the automator will ask if you wish to test the changes.

If you enter **y** or **Y**, then the function will exit and you will be free to build and test the cherry picked changes. It will also report the following:

```
When done testing, submit the backport by running 'bpr -s LPS-12345'
```

Once you are satisfied that the changes are working properly, you can simply run ```bpr -s LPS-12345``` and the current branch will be pushed through the BPR workflow.

If you enter **n** or **N**, then the function will continue with sending the pull request and progress the BPR ticket through the workflow.

### During Backport Workflow

When progressing the current branch through the backport workflow, it will prompt you for two items: The reviewer's GitHub and Jira usernames. The prompts will look similar to the following:

```
Enter the reviewer's GitHub name and press [ENTER] (jonathan.mccann): 
Enter the reviewer's Jira name and press [ENTER] (jonathanmccann): 
```

The values within the parenthesis are the default values you set for ```DEFAULTGITHUBREVIEWER, and DEFAULTJIRAREVIEWER```. If you press enter without entering any text, the default usernames will be used. If you wish to send the pull request to a different user, then you are able to manually type in their usernames.

## Potential Issues

Since the function uses commit messages to find the necessary commits to backport, if another commit contains the LPS number, it will also be cherry picked. Each commit that is cherry picked is shown, so please be sure that only the commits you want are included in your branch.