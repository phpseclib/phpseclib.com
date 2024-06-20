---
id: phpbb
title: phpBB MOD Text Template
---

If you're going to be applying code changes by hand the [various diff formats](https://en.wikipedia.org/wiki/Diff#Variations) are of limited usefulness. They're great for automated tools like [patch](https://en.wikipedia.org/wiki/Patch_(Unix)) but not so much for by hand edits.

A more useful format for by hand edits is the phpBB MOD Text Template. Unfortunately, as of [April 3, 2022](https://www.phpbb.com/community/viewtopic.php?f=14&t=2618701) the phpBB wiki wherein the format was described is no more, hence this page, which has been [adapted from archive.org](https://web.archive.org/web/20110621071313/https://wiki.phpbb.com/MOD_Text_Template).

---

The phpBB Modification Text Template is the former method used for installing MODs. It is a template for a text file containing instructions how to install a phpBB Modification. The text template is strictly for use with phpBB 2.0 modifications. For phpBB 3.0 modifications, please use the MODX template.

This article is for historical reference, as phpBB 3.0 modifications require the MODX format. The text template actions can however be useful for quick instructions (for instance, in a post to another user explaining what said user needs to do).

## Basic Template

The basic template is listed in the basic example. There are two important parts that can be separated, the meta information called the Header, and the installation instructions called the actions.

The text template is available in English only, for producing modifications instructions in other languages use [MODX](https://web.archive.org/web/20061029120010/http://www.phpbb.com/mods/modx/modx-part3.pdf).

```
##############################################################
## MOD Title: mod_title_goes_here
## MOD Author: your_phpBB_username < your_email > (your_real_name) your_url
## MOD Description: some_description_goes_here
## MOD Version: x.x.x
##
## Installation Level: (Easy/Intermediate/Advanced)
## Installation Time: x Minutes
## Files To Edit: file_1,
##      file_2,
##      file_3,
##      etc
## Included Files: (N/A, or list of included files)
## License: http://opensource.org/licenses/gpl-license.php GNU General Public License v2
##############################################################
## For security purposes, please check: http://www.phpbb.com/mods/
## for the latest version of this MOD. Although MODs are checked
## before being allowed in the MODs Database there is no guarantee
## that there are no security problems within the MOD. No support
## will be given for MODs not found within the MODs Database which
## can be found at http://www.phpbb.com/mods/
##############################################################
## Author Notes:
##
##############################################################
## MOD History:
##
##   YYYY-MM-DD - Version x.x.x
##      - version notes go here
##
##############################################################
## Before Adding This MOD To Your Forum, You Should Back Up All Files Related To This MOD
##############################################################

#
#-----[ ACTION ]------------------------------------------
#

#
#-----[ ACTION ]------------------------------------------
#

#
#-----[ ACTION ]------------------------------------------
#

#
#-----[ SAVE/CLOSE ALL FILES ]------------------------------------------
#
# EoM
```

## Header
The header contains all the important meta information about a modification including it's title, description, author, version, and important installation notes (called author notes). Below is a brief outline of each header field.

### Title
**Required** The title is the name of the modification. Choose a good name and keep the version number out of it.

### Example 2.1. Example usage of title.

```
## MOD Title: Example Modifications Team MOD
```

### Author
**Required** The author is the person/people who wrote the modification. Multiple authors can be included using multiple author fields. The following information can be provided about an author

Optional fields may be replaced with 'N/A'.

### phpBB.com Username
**Required** You are required to provide your phpBB.com username. If a previous author working on the MOD does not have one you can leave as N/A.

### E-mail Address
**Optional** The author's e-mail address. You may optionally provide an e-mail address, and may optionally obscure it.

### Real Name
**Optional** You may optionally provide your real name.

### URL
**Optional** You may list your website homepage.

### Examples
#### Example 2.2. Examples of allowed usage

```
## MOD Author: wGEric < myemail@mysite.com > (Eric Faerber) http://mods.best-dev.com

## MOD Author: wGEric  < N/A > (Eric Faerber) http://mods.best-dev.com

## MOD Author: wGEric < myemail@mysite.com > (N/A) http://mods.best-dev.com

## MOD Author: wGEric < N/A > (Eric Faerber) N/A

## MOD Author: wGEric < N/A > (N/A) N/A
```


#### Example 2.3. Examples of wrong usage.

```
## MOD Author: N/A < myemail@mysite.com > (Eric Faerber) http://mods.best-dev.com

## MOD Author: wGEric <  > (Eric Faerber) http://mods.best-dev.com

## MOD Author: wGEric <  > () http://mods.best-dev.com

## MOD Author: wGEric <  > () 

## MOD Author:  < myemail@mysite.com > () http://mods.best-dev.com
```


### Multiple Authors
A modification can have more than one author. Each author will need his or her own line and all of them must be filled out correctly. It is suggested that authors are listed chronologically with the top author being the most recent.

#### Example 2.4. Example Multiple MOD Author usage

```
## MOD Author: wGEric < N/A>> (Eric Faerber) http://mods.best-dev.com/
## MOD Author: battye < user@domain.tld > (N/A) http://www.cricketmx.com/
## MOD Author: evil<3 < N/A > (N/A) N/A
## MOD Author: Highway of Life < N/A > (N/A) http://www.startrekguide.com
```

### Description
**Required** This is the description of the MOD. It can be short or long. It is preferred that you keep this short and use the Author Notes section to provide detailed information. This can span multiple lines (each line must begin with ##).

### Version
***Required** This is the version of the MOD. phpBB uses the [Linux Kernal Numbering System](https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch01s03.html).

### Installation Level
**Required** The installation level is the level of difficulty you think your MOD is to install. While it is up to the MOD Author to determine what level of difficulty a MOD is, the modifications team may decree that a MOD is more difficult than listed on occasion (only if we thought it was hard when it was labelled as easy). Valid values are contained in the set {'Easy', 'Intermediate', 'Hard'}, with other values being invalid. The level of difficulty expected is outlined below (not really enforced):

_Easy_ Easy to install MODs make a few small edits to phpBB and maybe an SQL query. Usually limited to the set of edits {'OPEN', 'FIND', 'AFTER, ADD', 'BEFORE, ADD', 'REPLACE WITH'}, but may use limited IN-LINE actions or SQL. Easy MODs will only take a few minutes to install typically.

_Intermediate_ Intermediate MODs require a bit more from the user in understanding editing files and running SQL, but are still capable of being installed by people with no formal training. Usually involves more extensive use of IN-LINE and SQL actions, and may involve setting permissions. Intermediate MODs will typically take longer to install than easy MODs.

_Hard_ The hard level can be deceiving and was once also referred to as advanced which is possibly more correct. Nothing is hard if you know how to do it. However these MODs may be hard for those with no formal training in understanding file and database systems. They may make use of the full feature set of the MOD Template. If your MOD falls in this category you may want to rethink about it's architecture to make it easier to use for less experienced users. Hard MODs may take more than an hour to install and configure.

As these ratings are rather subjective the MOD Team will rarely argue with a MOD Author over their rating. The above list is merely a suggested guideline and not absolute.

```
## Installation Level: Intermediate
```

### Installation Time
**Required** Estimate how long it will take an average user to install your MOD in minutes. A good way to choose is to install the MOD yourself by hand as if a user were to install it themselves. Keep in mind you will probably be able to install it in less time than the average user can. Again as this is subjective the MOD Team will only question it if the estimate is grossly unreasonable, say for example 50 edits in 5 minutes. Be sure to allow time for edited files to be uploaded by the user.

Providing an estimate is more of a courteous gesture to potential users of the MOD so they can judge whether they will attempt to install (or if they have time to install).

```
## Installation Time: 25 Minutes
```

### Files to Edit
**Required** Collect a list of all files you have edited (used the OPEN action on) and list them here. If none are edited use N/A. Files are relative to the root phpBB path. Do not include a file count, only list the files (though the requirements for this field are generally relaxed).

```
## Files to Edit: N/A

or

## Files to Edit: posting.php
## includes/bbcode.php
```

### Included Files
**Required** A listing of all files that are included as part of the MOD as copied in the copy action in the MOD template file. If none are to be copied, use N/A. Files are relative to the MOD template file path. Do not include a file count, only list the files (though the requirements for this field are generally relaxed).

```
## Included Files: root/display.php
##   root/includes/functions_mod_display.php
```

### License
**Required** Provide licensing information about your MOD. 99% of the time this will be legally required to be the GNU GPL.

```
## License: http://opensource.org/licenses/gpl-license.php GNU General Public License v2
```

### Security Disclaimer
**Required** The security disclaimer is an important piece of information that informs users of their obligation to securing their own boards. This is required for all submissions to the phpbb.com modifications database. It simply states to check for updates to the MOD at phpbb.com before installing it. This helps users obtain the most up-to-date, bug free and secure version of your MOD available at the current time.

It also contains support instructions, stating that phpbb.com will be provide support if the MOD was obtained elsewhere than phpbb.com.

We do check MODs for security and other problems before we allow any MOD within our MOD Database. This is to provide security for those that use these MODifications. Although we do check for security problems, we can't guarantee that there aren't any within the MODs found in our MOD Database. After all, we are human and make mistakes. If a security exploit is found within one of the MODs within the MOD Database, we will take whatever action we feel is necessary.

```
## For security purposes, please check: http://www.phpbb.com/mods/
## for the latest version of this MOD. Although MODs are checked
## before being allowed in the MODs Database there is no guarantee
## that there are no security problems within the MOD. No support
## will be given for MODs not found within the MODs Database which
## can be found at http://www.phpbb.com/mods/
```

### Author Notes
**Required*** The author notes allow you to display extra notes to your users before they install the MOD. These may include PHP requirements beyond those of the base phpBB package, extra libraries required, and any other notes that your users need to be aware of before attempting installation of your MOD.

*_Although the author notes field is required, it may be left blank if you have nothing to write here._

```
##############################################################
## Author Notes: Make sure to backup your forum and it's database before proceeding
## Thanks to the many users that assisted during development.
##############################################################
```

### MOD History
**Optional** The history section allows you to tell your users a little bit about the revision history of your MOD. While it isn't required, it helps users keep track of updates to your MOD by providing a list of updates made in each release. If you are going to use this field you must use the following convention for entries to the history field. You may list in either date ascending or descending order.

```
##  YYYY-MM-DD - Version x.y.za
##   - version notes go here
##   - More version notes can go on new lines
```

Where YYYY is the year (4 digits), MM is the month (2 digits, leading zero), and DD is the day of the month (2 digits, leading zero). You can each change on a new line one after each other using the dash `-` character as a bullet.

### Installation Disclaimer
**Required** The installation disclaimer is a small piece of text that reminds users of your MOD to practise safe modification methods on their forum, relating specifically to backing up before installing. Neither phpBB, nor is the modification author responsible for data loss caused by the installation by any MOD.

```
## Before Adding This MOD To Your Forum, You Should Back Up All Files Related To This MOD
```

## Actions
After providing a bit of information about your MOD to the users, you provide a complete list of actions to do to successfully install the MOD. There are a couple of different actions that you can instruct users to do as listed below. The convention is that actions are case sensitive and must be written in uppercase.

- SQL
- COPY
- DIY INSTRUCTIONS
- OPEN
- FIND
- REPLACE WITH
- AFTER, ADD
- BEFORE, ADD
- INCREMENT
- IN-LINE FIND
- IN-LINE REPLACE WITH
- IN-LINE AFTER, ADD
- IN-LINE BEFORE, ADD
- IN-LINE INCREMENT

### Action definition
All actions must be defined using a standard syntax. An action consists of the name of the action (what to do), optionally a comment, and how to do it (for example, open which file, find what text?). A translation of this is that the action is a verb, and the 'how to do it' is a noun. Shown below is what a typical action will look like.

```
#
#-----[ ACTION ]------------------------------------------
#
do stuff here
#
#-----[ ACTION ]------------------------------------------
#
# The comment goes here
#
do stuff here
```

Comments can span as many lines as you wish, but always start with a hash `#`. The hash is always the first character and must line up in contiguous with previous hashes that make up the action definition.

### The SQL Action
The SQL action tells the person who is installing the MOD that they need to execute the SQL queries. The SQL queries need to use the default table prefix (phpbb_), be written for MySQL, and end in a semicolon ( ; ). You can put multiple queries under the same SQL action.

#### Example 2.5. Example SQL action usage

```
#
#-----[ SQL ]------------------------------------------
#
INSERT INTO phpbb_config ( config_name, config_value ) VALUES ('foo', 'bar');
```

#### Example 2.6. Another example SQL action usage

```
#
#-----[ SQL ]------------------------------------------
#
INSERT INTO phpbb_config ( config_name, config_value ) VALUES ('foo', 'bar');
INSERT INTO phpbb_config ( config_name, config_value ) VALUES ('foo2', 'bar2');
```

If you have a script that the user can use to perform the SQL queries for them, you need to use the DIY INSTRUCTIONS action to tell the user to execute that script. You can't put under the action that the user needs to use the update script or have a comment with nothing under the action. Do not use this action if you want users to use the update script instead of executing the SQL manually.

### The COPY Action
The COPY action is used to copy files from the MODs package to the board the MOD is being installed on. Each file to be copied needs to have its own line that starts with copy and has to in the middle. The part after copy and before to is the file in the MODs package. It needs to be the path from the root of the MODs package. The part after the to is where the file will go and be named from the root of the phpBB install. You must put a filename for both parts in each copy line.

You can use wild cards `*` to copy multiple files without having to list all of them. See Example 2.7, “Example COPY action usage” to see how to properly use wild cards.

#### Example 2.7. Example COPY action usage

```
#
#-----[ COPY ]------------------------------------------
#
copy foo.php to foo.php
copy includes/bar.php to includes/bar.php
copy images/*.gif to images/*.gif
copy files/*.txt to files/*.txt
copy cache/*.* to cache/*.*
```

### The DIY INSTRUCTIONS Action
DIY INSTRUCTIONS are instructions that the person installing the MOD will have to do before the MOD can fully be installed. DIY means 'Do it yourself.' These instructions can include running a script that executes SQL queries, CHMOD files, or anything else. Typically these actions should be shown at the end of your MOD as post installation instructions.

#### Example 2.8. Example DIY INSTRUCTIONS action usage

```
#
#-----[ DIY INSTRUCTIONS ]------------------------------------------
#
Now if your on linux CHMOD foo.php and bar.php to 777
Then point your web browser db_install.php and follow the instructions presented.
```

### The OPEN Action
The OPEN action is used to open a file for editing. An OPEN action needs to be done before any FIND or other actions that edit a file. The file name needs to have the path from the root of phpBB and is case sensitive. All slashes need to be forward (`/` not `\`) and no slash is to be at the beginning of the file name/path.

> **Note**
>
> The OPEN action must be followed by a FIND action.

#### Example 2.9. Example OPEN action usage

```
#
#-----[ OPEN ]------------------------------------------
#
templates/subSilver/viewtopic_body.tpl
```

### The FIND Action
The FIND action is used to find certain code within a file so that it can be edited or have more code added around it. FINDs need to be unique so that the edits are done at the right spot. FINDs can be partial and don't have to be at the beginning of the line.

You can perform multiple actions on one FIND. So, you can do a FIND and then on that FIND do a BEFORE, ADD, AFTER, ADD, IN-LINE actions, and a REPLACE WITH.

> **Note**
>
> The FIND action must be followed by a `FIND`, `REPLACE WITH`, `AFTER, ADD`, `BEFORE, ADD`, or an `IN-LINE FIND` action.

#### Example 2.10. Example FIND action usages

```
#
#-----[ FIND ]------------------------------------------
#
$lang['General'] = 'General Admin';
```

#### Example 2.11. Example partial FIND action usages

You can also do partial FIND actions as follows:

```
#
#-----[ FIND ]------------------------------------------
#
'General Admin'
#
#-----[ FIND ]------------------------------------------
#
Admin';
#
#-----[ FIND ]------------------------------------------
#
$lang['General'] =
```

As you can see, the code that is under the FIND isn't the whole line and comes from different parts of the line. Although the FIND is partial, the whole line will be what the rest of the actions will work with. If you want to work code within a line, the IN-LINE actions are what you will need to use.


### The REPLACE WITH Action

The REPLACE WITH action is used to replace code with different code. This action needs to be preceded with a FIND action and everything within the FIND action will be replace with this action.

As there is no action to delete code in the MOD Template, a delete is just defined as replace with blank. You can use the REPLACE WITH action to either comment out the code, or to replace it as blank.

We recommend that you avoid using the REPLACE WITH action where avoidable. If you can, use IN-LINE actions or any others that would work instead to increase compatibility with other MODs.

#### Example 2.12. Example REPLACE WITH action usage

```
#
#-----[ REPLACE WITH ]------------------------------------------
#
$lang['General'] = 'New string';
```

### The AFTER, ADD Action
The AFTER, ADD action is used to add code to the file. It must be preceded by a FIND action because it adds the code after the line(s) that have been found in the FIND action. If you want to add code after a certain part within one line, you need to use IN-LINE actions. This action will only add code after the line(s) that was found in the FIND.

#### Example 2.13. Example AFTER, ADD action usage

```
#
#-----[ AFTER, ADD ]------------------------------------------
#
if ( $userdata['user_level'] == ADMIN )
{
  edit_user();
}
```

### The BEFORE, ADD Action

The BEFORE, ADD action is exactly like the AFTER, ADD action except it adds the code before the line(s) that were found in the FIND action. If you want to add code before a certain part within one line, you need to use IN-LINE actions. This action will only add code before the line(s) that was found in the FIND.

#### Example 2.14. Example BEFORE, ADD action usage

```
#
#-----[ BEFORE, ADD ]------------------------------------------
#
if ( $userdata['user_level'] == USER )
{
  redirect('index.'.$phpEx);
}
```

### The INCREMENT Action
The INCREMENT action is used to add (increment) or subtract (decrement) to numbers. This is most useful for colspans or rowspans in the template.

The INCREMENT action needs to be preceded by a FIND or an IN-LINE FIND, you need to replace the number you want to add or subtract to with {%:1}. If you have multiple numbers within the line that you want to add or subtract to, use {%:2}, {%:3}, and so on.

In the INCREMENT action, you say which one of the numbers in what is in the FIND you want to add or subtract to and then how much you want to add or subtract to. To add, use + and to subtract, use -. You can put nothing after you say which number you want to increase and it will be the same as +1. Here are some examples.

#### Example 2.15. Example INCREMENT action usage

```
#
#-----[ FIND ]------------------------------------------
#
Powered by <a href="http://www.phpbb.com/" target="_phpbb" class="copyright">phpBB</a> &copy; 2001, {%:1} phpBB Group<br />{TRANSLATION_INFO}</span></div>
#
#-----[ INCREMENT ]-------------------------------------
#
%:1 +10
```

#### Example 2.16. Other example INCREMENT action usages

```
#
#-----[ INCREMENT ]-------------------------------------
#
%:1

#
#-----[ INCREMENT ]-------------------------------------
#
%:2 -3

#
#-----[ INCREMENT ]-------------------------------------
#
%:3 +5
```

> **Note**
>
> You can use IN-LINE INCREMENT as well which is the same as above (except follows a IN-LINE FIND action).

### The IN-LINE FIND Action
IN-LINE FIND is used to find a specific part to one line so that you can do edits within the line. IN-LINE FIND must be preceded by a FIND, INCREMENT, REPLACE WITH, AFTER,ADD or BEFORE ADD action. All IN-LINE actions need to have an IN-LINE FIND preceding them.

#### Example 2.17. Example IN-LINE FIND action usage

```
#
#-----[ FIND ]------------------------------------------
#
$lang['Admin'] = 'General Admin';

#
#-----[ IN-LINE FIND ]------------------------------------------
#
General
```

### The IN-LINE AFTER, ADD Action
The IN-LINE AFTER, ADD action adds code within the same line but after what was found in the IN-LINE FIND without having the replace the whole line. IN-LINE AFTER, ADD must be preceded by an IN-LINE FIND (or other IN-LINE edit) and can only be one line. You cannot add multiple lines using IN-LINE AFTER, ADD.

#### Example 2.18. Example IN-LINE AFTER, ADD action usage

```
#
#-----[ IN-LINE AFTER, ADD ]------------------------------------------
#
phpBB
```

### The IN-LINE BEFORE, ADD Action
The IN-LINE BEFORE, ADD action is just like the IN-LINE AFTER, ADD action except it adds code before what was found in IN-LINE FIND. Just like IN-LINE AFTER, ADD, you must precede IN-LINE BEFORE, ADD with an IN-LINE FIND (or other IN-LINE edit) and it cannot be used to add multiple lines.

#### Example 2.19. Example IN-LINE BEFORE, ADD action usage

```
#
#-----[ IN-LINE BEFORE, ADD ]------------------------------------------
#
phpBB
```

### The IN-LINE REPLACE WITH Action
IN-LINE REPLACE WITH is used to replace all of the code that was found in IN-LINE FIND with what is in the IN-LINE REPLACE WITH. Like the other IN-LINE actions, IN-LINE REPLACE WITH must be preceded by an IN-LINE FIND (or other IN-LINE edit) and can only be one line. You cannot do a replace that is more than one line.

#### Example 2.20. Example IN-LINE REPLACE WITH action usage

```
#
#-----[ IN-LINE REPLACE WITH ]------------------------------------------
#
Common
```

### Conclusion
That is it for the list of available actions that you can use. You cannot make up any new actions to use in your MOD. You should be able to do what you want with the above actions.

### Concluding a MOD Text Template file
To conclude a text template installation file you always end it with the SAVE/CLOSE ALL FILES action. This action is special in that it has one mandatory comment, and may not act upon a noun. The exact syntax to be used if shown in Example 2.21, “How to end a text template example”. If anyone is wondering EoM is an initialism for End of MOD.

This action instructs the user to save all their edits and publish them to their website, pretty simple, but it just finishes off the MOD.

#### Example 2.21. How to end a text template example

```
#
#-----[ SAVE/CLOSE ALL FILES ]------------------------------------------
#
# EoM
```