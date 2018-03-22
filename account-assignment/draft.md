# Interfaces as Assistants

### Table of Contents
[Motivation](#motivation)\
[Interface by Forms](#interface-by-forms)\
[Interface as an Assistant](#interface-as-an-assistant)\
[Theory of Interfaces](#theory-of-interfaces)

### Motivation

I remember someone made an observation about websites that went along the lines of "the only way a person would need to interact with a website is through forms". I think that there is truth to that observation in quite a literal sense, but I actually want to explore this observation further because I've actually been trying build a theory of interfaces from it in order to have a unified design language across an application that I'm working on.

### Interface by Forms

Suppose you are in an email client and you want to affect the application by deleting three emails named e1, e2, e3. Normally what you would do is select the three emails by checking the corresponding checkboxes then clicking on the trash icon. However, you can imagine that another way to cause the same affect in the application is to pull out the hypothetical delete-email form, as shown

```
Delete Emails Form:
  emails(comma delimited):_____
```

and fill in the emails line with 'e1, e2, e3,' then press the go button. So for relatively simple affects like deleting emails, we see that we can actually replace the interface with a form and the user will still be able to use the application as wanted.

But even for a more elaborate affect, say if you are in a music player and you want to add a song to your current playlist. The most obvious way to do it is to press the plus icon (add to playlist button) beside the song, or drag the song into the 'playlist area'. Similarly, you can also pull out the hypothetical add-to-playlist form and fill in the name of the song that you want to add.

Another compelling reason why I think that all interfaces have an equivalent form version because that is actually the only way that it can be implemented. That in the sense that for every single affect on the application, server and front-end come to an agreement on a form, which then will be submitted will be processed by server and will cause the effect on the application.

### Interface as an Assistant

If we pretend that it is plausible for all UI to be replaced with forms, that begs the question "why do we have really elaborate interfaces anyways." I think that a possible way to look at is that elaborate interfaces act as assistants which help users fill in the forms more efficiently to affect the application.

A good way to illustrate why I think this idea might have some truth to it is by looking at the scenario of moving a file into a folder. Suppose we have the following file structure.

```
/home
  /my-folder
  my-file
```

Suppose I would like to move the file my-file into the folder /my-folder. The most primitive way to do this is to run a terminal in the location of /home and use the command

```
>>> mv my-File my-folder/my-file
```

This command calls the move executable (mv) and tells it to move the file my-file to the location my-folder/my-file. This is synonymous to pull out the move-file form and filling in the right parameters.

```
Move File Form:
  file: _____
  to: _______
```

If you are in the file explorer on your computer, to if you want to move my-file into /my-folder what you would probably do is drag the icon that represents my-file onto to icon that represents /my-folder. However, since we know that under the hood the file is actually moved by the command mentioned earlier.

I think that we can interpret it as follows, when you press on and start to drag the my-file icon, then computer program opens the move-file form and fills in the file: field with the string my-file, then when you move it over the hover over the /my-folder icon it fills in the to: field with the string /my-folder/my-file and finally when you drop the icon it presses the go button.

Similarly if you want to create the same affect but you are on a mobile phone,  you would probably open the mobile file explorer and navigate to the location of my-file, then select my-file and press the move button, the program then places you back to the home directory, and then you navigate to the folder you want to move the file into and press on it to move the file. You can imagine that while you are working through the process that in the background the program fills in the move-file form for you.

### Theory of interfaces

If we can accept that all interfaces can be reduced to a form, and that user interfaces are assistants that help you fill in forms to affect the application, then it is reasonable to argue that an effective interface is one the helps you fill in forms really well. How the interface helps you fill in the forms is probably a more complicated topic, however I think that some very desirable characteristics of form filling assistants (interfaces) is that they give you relevant information that will help you make decisions on how to fill the form, and that they provide useful information on the state of the form you are filling.

To come back to the example of deleting emails in an email client, I would argue that perhaps this interface\
[image here]()
because I think that the interface gives you information that you would like to know when you are trying to delete emails, such as email names, and timestamps, and it also helps you keep track of the state of the form in the sense that it shows you which emails are put into the delete-email form by highlighting the emails that you selected.

Which is a rather long winded way of saying that the UI oh the account assignment has the qualities of UI that I theorize are important but the same thing can be done with a simpler interface that is easier to program and test, important parts being that its easier to program and test.

[image here]()
