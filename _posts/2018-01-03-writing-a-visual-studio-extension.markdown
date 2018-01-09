---
layout: post
title:  "Writing a Visual Studio Extension"
date:   2018-01-03 20:02:15 +0000
categories: jekyll update
---

The git integration within Visual Studio has its critics, but I find it generally does most of the
things I need it to do in my day to day work. However, the lack of an ability to prune has been something
of an inconvenience. I know that I can configure git to prune on fetch, but I saw this as an opportunity
to experiment with building a basic extension to Visual Studio.

# Creating the project

![New project](/assets/vs-new-vsix-project.png)

First things first, we need to create a new extension project. This can be done by clicking the new project
button, picking a language of choice, choosing 'Extensibility' as the project type, and selecting 'VSIX Project'. This will create a new
project using a template that brings in all things necessary to get started on our extension.

# Adding a button

We are making an extension to run the git prune command on the current solution within visual studio, so there needs
to be some UI option to trigger this. For my extension I have utilised my limited drawing ability to create the following
image for my button (its a leaf because... prune):

![Git prune](https://github.com/Simon-PJ/Git-Prune-VS-Extension/blob/master/GitPrune/Resources/PruneCommand.png?raw=true)

This image now needs a button to display itself on. First off, navigate to the add new item dialog for the project. From here
select 'Custom Command'. Give it a relevant name and add it to the project.

![Add command](/assets/vsix-new-command.png)

Once this has been added to the project, go ahead and open up the .vsct file. In here there will be a load of XML.
To get a button visible there are a few things we need to add. First off navigate to the 'Groups' element. Inside
this element add a group along the lines of:

```
<Group guid="guidPrunePackageCmdSet" id="PruneToolbarGroup" priority="0xF000">
    <Parent guid="guidSHLMainMenu" id="IDM_VS_TOOL_PROJWIN"/>
</Group>
```

The group name and id will need to be used later on. The 'Parent' element id value specifies where
the button should be displayed. The value 'IDM_VS_TOOL_PROJWIN' will place it in the solution explorer toolbar. A
more thorough list of various Visual Studio toolbar ids can be found [here](https://docs.microsoft.com/en-us/visualstudio/extensibility/internals/guids-and-ids-of-visual-studio-toolbars).

Next, scroll down to find 'Buttons'. Inside this we want to add a 'Button' child element.

```
<Button guid="guidPrunePackageCmdSet" id="PruneButtonId" priority="0x0100" type="Button">
    <Parent guid="guidPrunePackageCmdSet" id="PruneToolbarGroup" />
        <Icon guid="guidImages" id="prune" />
        <Strings>
            <ButtonText>Git Prune</ButtonText>
        </Strings>
    </Button>
</Buttons>
```

Note here that both Button's and Parent's guid is the same as that from Group, and Parent's id is the same as Group's.

The Icon element and its attributes are the next area of focus. These determine what will be shown on the
button - in this case, my delightful leaf. We are going to have to amend a couple more XML elements, but first
we need to add the image to the project.

I took a shortcut at this point. The template should have added a png to the Resources folder. Instead of
adding my own image, I simply went and drew my leaf over the top of the first icon in there, deleting the rest.
This leaves a long strip of empty image, but this was coming dangerously close to becoming design work.

Back to the XML; we need to be in the 'Bitmaps' element. If like me you have drawn over the existing image,
the existing child element can be replaced in here with:

```
<Bitmap guid="guidImages" href="Resources\Prune.png" usedList="prune"/>
```

Where href should be the location of the image within the assembly. The guid attribute should be the same as
Icon's in the previous section, and usedList should be the same as Icon's id.

One more step to get a nice leafy button in your solution explorer. This time we want to be in the 'Symbols'
element. In here add the following:

```
<GuidSymbol name="guidPrunePackageCmdSet" value="{526f2e07-251d-45c9-b5bc-1d26a216d505}">
    <IDSymbol name="PruneToolbarGroup" value="0x0190"/>
    <IDSymbol name="PruneButtonId" value="0x0180"/>
</GuidSymbol>
  
<GuidSymbol name="guidImages" value="{8515b00c-7416-45aa-8a64-b4be2cb71303}" >
    <IDSymbol name="prune" value="1" />
</GuidSymbol>
```

The first element is related to the button. The two symbols should tie in with the guid and id attributes we
added back in the first section on the Group element.

The second GuidSymbol block is about defining different icons within the icon image (in my case Prune.png).
The name of the GuidSymbol should reflect the guid in the Bitmap element. IDSymbol should provide a name
consistent with Bitmap's usedList (more icons can be added by appending to usedList and specifying them
as seperate IDSymbols). For the value attribute we need to cast our minds back to the origins of the icon
image. There were a number of icons in here to begin with. The number we put in the value attribute represents
where in that sequence our desired image sits.

After all that, if you run the solution an instance of Visual Studio should start. Once it loads you should see
a leafy button in the solution explorer.

# Adding functionality

Adding a button to Visual Studio is all well and good and fairly satisfying in its self, but to be actually
useful it should do something. What we want is for clicks of the prune button to try and prune our current solution.
If there is no current solution, or there is no git source control in the current solution, that should be handled.

When we added a new Custom Command through the add item dialog, a .cs file will have been created with whatever
name the command was given. Open this and go to the constructor. There will be a line in here which instantiates
a new MenuCommand. Change this line to something similar to:

```
var menuItem = new MenuCommand(this.PruneCurrentGitDirectory, menuCommandID);
```

There will now be some compiler errors because the method PruneCurrentGitDirectory has not yet been created, so
we should do this next:

```
private void PruneCurrentGitDirectory(object sender, EventArgs e)
{
    var dte = (DTE)Package.GetGlobalService(typeof(DTE));

    if (string.IsNullOrEmpty(dte.Solution.FullName)) return;

    var solutionDirectory = Path.GetDirectoryName(dte.Solution.FullName);
    var gitDirectory = FindGitRepoDirectory(solutionDirectory);

    if (gitDirectory == null)
    {
        MessageBox.Show("No git repo found");
        return;
    }

    var displayString = RunPruneCommand(gitDirectory);

    MessageBox.Show(displayString);
}

private string RunPruneCommand(string solutionDirectory)
{
    var pruneCmd = $"/c cd {solutionDirectory}&git remote prune origin";

    var process = new System.Diagnostics.Process
    {
        StartInfo = new ProcessStartInfo
        {
            FileName = "CMD",
            WindowStyle = ProcessWindowStyle.Hidden,
            Arguments = pruneCmd,
            RedirectStandardOutput = true,
            UseShellExecute = false,
            CreateNoWindow = true
        }
    };

    process.Start();

    var output = process.StandardOutput.ReadToEnd();
    var displayString = string.IsNullOrEmpty(output) ? "Nothing to prune" : output;

    return displayString;
}

private string FindGitRepoDirectory(string directory)
{
    if (Directory.Exists($"{directory}/.git"))
    {
        return directory;
    }

    var parentDirectory = Directory.GetParent(directory);

    if (parentDirectory != null)
    {
        return FindGitRepoDirectory(parentDirectory.FullName);
    }

    return null;
}
```

I've finished on a decent chunk of code but it isn't too complex. Essentially, after clicking the prune button
the code will grab the current solution directory and travel upwards to try and find a git repo. If it doesn't find
one a message will be displayed, if it does, git prune will be ran against the remote repository.

Go ahead, fire up the project and reap the rewards.

![Git prune in VS](/assets/git-prune-in-vs.png)

Outstanding.

# In summary

The end product of all this is a small extension for Visual Studio that can prune your repositories via a
button click. Not terribly complicated, but it serves as a good introduction to writing extensions.

Extensions can be added to the Visual Studio marketplace. I've published this one, which can be
found under the descriptive name of 'Git Prune Button'.