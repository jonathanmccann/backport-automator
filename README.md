# Backport Request Automator

## Features

* Cherry pick commits based off of LPD number
* Send the pull request

## Prerequisites

#### Hub

Hub is a wrapper for Git that is tailored for GitHub. Hub makes it much simpler to send pull requests. To set up Hub please visit - https://hub.github.com/

#### Bash functions

Copy the functions from the [bpr](https://github.com/jonathanmccann/backport-automator/blob/master/bpr) file to your ```.bash_aliases``` file so that you are able to call the functions from the command line.

At the start of the ```bpr``` file there are two configurable values:

```CURRENTUSERGITHUBNAME``` and ```DEFAULTGITHUBREVIEWER```

For the current user value, enter in your GitHub username. For the reviewer value, enter in the GitHub username of the reviewer you normally send backport requests to.

Note that the reviewer value is simply the default. If you wish to submit the backport to a different reviewer, you will be able to specify their name on the fly.

## Usage

### Preparing to Backport

This function assumes that you are backporting to a quarterly release branch (eg **release-2023.q4**). Other versions are not supported at this time.

To begin, be sure that your **EE master branch** is up to date and contains the fix that you wish to backport. There is no need to create a new branch as the function does that for you.

### Beginning Backport

The basic command is ```bpr LPD-12345 release-2023.q4```. Running this will check out a new branch, cherry pick commits that have ```LPD-12345``` in their commit message, push the branch to your origin, and send the pull request.

However, cherry picking can be tricky and some times there are conflicts. When that happens, the function will exit with one of the following messages:

```
Could not perform cherry pick for 3706ecc7c2f2 LPD-12345 Commit Message
Please resolve the conflict(s) and then run 'bpr -s LPD-12345 release-2023.q4'
```

or

```
Could not perform cherry pick for 3706ecc7c2f2 LPD-12345 Commit Message
Please resolve the conflict(s) and then run 'bpr -h LPD-12345 release-2023.q4 3706ecc7c2f2'
```

When a cherry picking error is occurred, you must manually resolve the conflict and then run the provided command. If there are more conflicts then the above will repeat.

Once all of the commits have been successfully applied, then you are able to test your changes or submit the backport.

There are two distinct flags that are used and seen in the above messages:

##### Submit Flag

The ```-s``` flag signifies to the function that the current branch is done cherry picking and ready to be submitted for review. Note that the function will not ask if you wish to test the changes, so please do so before running this command.

It requires two arguments - the **LPD number** and the **quarterly release branch name**.

##### Hash Flag

The ```-h``` flag signifies to the function that there are more commits that need to be cherry picked. It will then begin cherry picking on the specified commit hash. If there are no more conflicts, it will then submit the current branch for review.

It requires three arguments. The first argument is the **LPD number**, the second argument is the **quarterly release branch name**, and the third argument is the next **commit hash** that the function needs to cherry pick.

### After Cherry Picking Has Finished

Once the cherry picking has finished, the automator will ask if you wish to test the changes.

If you enter **y** or **Y**, then the function will exit and you will be free to build and test the cherry picked changes. It will also report the following:

```
When done testing, submit the backport by running 'bpr -s LPD-12345 release-2023.q4'
```

Once you are satisfied that the changes are working properly, you can simply run ```bpr -s LPD-12345 release-2023.q4``` and the current branch will submitted as a pull request.

If you enter **n** or **N**, then the function will continue with sending the pull request.

### Submitting For Review

When submitting the current branch for review, it will prompt you for a single item: The reviewer's GitHub username. The prompt will look similar to the following:

```
Enter the reviewer's GitHub name and press [ENTER] (jonathanmccann):
```

The value within the parenthesis is the default value set for ```DEFAULTGITHUBREVIEWER```. If you press enter without entering any text, the default username will be used. If you wish to send the pull request to a different user, then you are able to manually type in their username.

## Potential Issues

Since the function uses commit messages to find the necessary commits to backport, if another commit contains the LPD number, it will also be cherry picked. Each commit that is cherry picked is shown, so please be sure that only the commits you want are included in your branch.