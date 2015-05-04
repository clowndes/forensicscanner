# Plugins #



## What is a "plugin"? ##
A plugin is a Perl script that follows a specific structure/format so that it can be used with the RegRipper suite of tools.  Not following this structure or format may mean that a plugin will not work, or work correctly, with the tools.

Plugins are located in the "plugins" directory beneath the RegRipper tools.  As the plugins are Perl scripts, they end with the ".pl" extension.  The plugins are accessed by the RegRipper tools via the _require_ pragma.  This allows the plugins to by dynamically loaded at run-time, rather than having to compile them ahead of time.

While the plugins are Perl scripts, they need to follow a structured format in order to function properly within the RegRipper framework.  This may seem to be a burden or unnecessary to some, but for most folks who would write plugins, this means that getting started writing a plugin is as easy as cut-and-paste.

## Variables ##
The tools that use the plugins make a number of variables available to the plugin.  Several of these variables help identify the version of the Windows operating system being scanned, while other provide reference information (i.e., drive letter, paths to hives, etc.).  These variables are stored in a Perl hash, and passed to the plugins as a reference when they call the _::getConfig()_ function.

_hostname_,_computername_ - The hostname and ComputerName values retrieved from the Registry within the mounted image.

_ProductName_,_CurrentVersion_,_CurrentType_,_CSDVersion_,_InstallDate_,_CurrentBuildNumber_,_SystemRoot_ - Values retrieved from the Registry within the mounted image.  Several of these values, particularly _CurrentVersion_, can be used to manage program flow within the plugins.

_osflag_ - The _CSDVersion_ value is run through a table (see below) to get this value, so that it can be used in logical AND or OR comparisons in order to manage program flow within the plugins.

```
my %os_table = ('5.1' => 1,
		'5.2' => 2,
		'6.0' => 4,
		'6.1' => 16
                '6.2' => 32);
```

_drive_ - The path to the system32 directory that is passed to the tool is parsed to get the "drive" letter.  Depending upon how the image is mounted, this may be a drive letter (i.e., F:\), or if a VSC is mounted, it may be a path (i.e., C:\image\vsc1).  Similarly, the value _systemroot_ is also provided, which is the path to the "Windows" directory within the mounted image.  **Note** that this is _not_ the _SystemRoot_ value (mentioned above) that is extracted from the Registry within the mounted image.

_system_,_software_,_sam_,_security_ - paths to the main hive files within the mounted volume.

_reportfile_,_logfile_ - paths to the reportfile and logfile for the scan; these are provided in the hash for use by the main tool, and do not need to be used by the plugins.

_userprofile_ - This value is not available to the plugins until the main tool begins scanning the user profiles.  This value can be appended with "NTUSER.DAT" to provide access to that hive, and then the _osflag_ value can be used to determine the proper location for the USRCLASS.DAT hive so that the plugins can access that hive, as well.

Again, each of these values is available to the plugins as a hash reference.  What this means is that the plugin can request the reference by calling the _::getConfig()_ function:

```
my $hash = ::getConfig();
```

From this point, the values can then be accessed as follows:

```
$hash->{software}
```

...and...

```
$hash->{userprofile}
```


## Plugin Header & Comments ##
Each plugin should contain a header, which should include the name of the plugin, a description of the purpose of the plugin, a change/revision history, and the name of the author, or whomever modified the plugin.  In order to be treated as a comment, each line should start with a pound symbol, or '#'.


## Package ##
Even though the plugins are Perl scripts that end in ".pl", they are essentially Perl modules.  As such, each plugin needs to be identified as a Perl package, which should be the same name as the plugin file itself, minus the ".pl".  For example, for the plugin named "acmru.pl", the _package_ line should read:

```
package acmru;
```

This line is the first line of actual executable code that is processed by the Perl run-time.  It can be the first line of the script, or can follow the header/comments, but it must be the first line of the actual executable code of the plugin.

## Plugin Configuration Hash ##
Every plugin contains a Perl hash (i.e., "%config") that contains configuration information about the plugin, that provides information about the plugin, and can be queried by external tools.  Tools can access the plugin configuration hash by getting the name of the plugin and then calling:

```
my $pluginconfig = $pkg->getConfig();
```

Notice that the _$pluginconfig_ value is now a reference to the configuration hash, and each of the below values will then be accessed via:

```
$pluginconfig->{_value name_}
```

_hive_ - a string value that identifies to which hive (i.e., SAM, Security, Software, etc.) the plugin applies.  This configuration value will be deprecated.

_hivemask_ - a flag value that identifies to which hive or hives the plugin applies (see table below).

_hasShortDescr_ - binary value that indicates if the plugin has a short description (**NOTE**: Every plugin should have a short description).

_hasDescr_ - binary value the indicates if the plugin has a long description.

_hasRefs_ - binary value that indicates if the plugin contains references, or links to information (i.e., MS KnowledgeBase articles) that add credibility to the plugin.

_osmask_ - a flag value that identifies to which Windows operating system versions the plugin applies (see table below).

_version_ - a value that indicates when the plugin was created/modified, in the format "YYYYMMDD".  Authors or anyone who updates or edits a plugin must update this value, and should provide comments in the header of the plugin that describe what changes were made and why.

_output_ - a string value that describes the form of output that the plugin writes.  This can be "tln" for timeline format output, "csv" for comma-separated value output, "report" for free-form reporting format, etc.

_category_ - this is a value that indicates to which category or categories (as part of timeline analysis) the plugin applies.

_type_ - the 'type' value identifies the type of plugin; for the RegRipper suite of tools, this will always be "Reg".  This value is really more for use within another project.

### osmask value ###
The "osmask" value within the plugin configuration hash indicates to which version(s) of Windows the plugin applies.  The following table demonstrates values

|Bit|7|6|5|4|3|2|1|0|
|:--|:|:|:|:|:|:|:|:|
|Value|128|64|32|16|8 |4 |2 |1 |
|OS Version|  |Windows Server 2012|Windows 8|Win7/Win2008R2|2008|Vista|2003|XP|

Within the RegRipper tools, an AND (i.e., "&") operation is used to determine the version(s) of Windows to which the plugin applies.  For example, an "osmask" value of 1 indicates that the plugin applies to Windows XP only, where an "osmask" value of 31 indicates that the plugin applies to versions of Windows from XP, up to and including Windows 7 and Windows 2008R2.

A reference for OS version numbers can be found [here](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724832%28v=vs.85%29.aspx).

### hivemask value ###
The "hivemask" value within the plugin configuration hash indicates to which hive(s) the plugin applies.

|Bit|7|6|5|4|3|2|1|0|
|:--|:|:|:|:|:|:|:|:|
|Value|128|64|32|16|8 |4 |2 |1 |
|Hive|  |  |USRCLASS.DAT|NTUSER.DAT|Software|System|SAM|Security|

Within the RegRipper tools, an AND (i.e., "&") operation is used to determine the hive(s) to which the plugin applies.

At the point, a plugin %config hash might appear as follows for a plugin intended for the NTUSER.DAT hive, specifically for Windows XP/2003 systems:

```
my %config = (hive => "NTUSER\.DAT",
	      hivemask      => 0x10,
              hasShortDescr => 1,
              hasDescr      => 0,
              category      => "User Activity",
              type          => "Reg",
              output        => "report",
              class         => 1,
              hasRefs       => 0,
              osmask        => 3,  #XP/2003
              version       => 20120823);
```

### Categories ###



## _Pluginmain()_ ##
All plugins contain a _pluginmain()_ function, which is called by external tools (RegRipper, rip.pl/.exe, etc.).  This function contains the primary executable code for the plugin.

Plugins can contain their own subroutines, or can pass variables to subroutines provided by the code calling them.  For example, RegRipper and rip.pl/.exe both provide the _getTime()_ function, which takes two 32-bit values that comprise a 64-bit [FILETIME](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724284(v=vs.85).aspx) object, and return a 32-bit Unix epoch time.  This function is accessible by all plugins by calling _::getTime($$)_.

## Final ##
Each plugin must return a value; as such, the final line of the plugin should be:

```
1;
```