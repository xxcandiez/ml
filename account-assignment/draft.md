# Interfaces as Assistants

### Table of Contents
[Motivation](#interface-by-forms)
[Interface by Forms](#interface-by-forms)
[Interface as an Assistant](#interface-as-an-assistant)

### Motivation <span id='1'></span>

I remember someone made an observation about websites that went along the lines of "the only way a person
would need to interact with a website is through forms". I think that there is truth to that observation in
quite a literal sense, and that the observation is worth exploring further because it might be possible to
build a theory of web interfaces from it.

### Interface by Forms

Suppose you are in an email client and you want to affect the application by deleting three emails named e1,
e2, e3. Normally what you could do is select the three emails by filling in the checkboxes on the correct
lines, then clicking on the trash icon. Another way to make the same affect is to pull out the hypothetical
delete-email form, and fill in the lines with the names e1, e2, e3.

Even for a more elaborate affect like if you are in a music player and you want to add a song to your current
playlist, perhaps you can press the plus icon beside the song, or drag the song into the playlist area.
Similarly, you can also pull out the hypothetical add-to-playlist form and fill in the name of the song that
you want to add.

Another compelling reason why I think that all interfaces have an equivalent form version because that is
actually the only way that it can be implemented. That in the sense that for every single affect on the
application, server and front-end come to an agreement on a form, which then will be submitted will be
processed by server and will cause the effect on the application.

### Interface as an Assistant <span id='3'></span>

If we pretend that it is plausible for all UI to be replaced with forms, that begs the question "why do we
have really elaborate interfaces anyways." I think that a possible way to look at is that elaborate
interfaces act as assistants which help users fill in the forms more efficiently to affect the application.

A good way to illustrate why I think this idea might have some truth to it is by looking at the scenario of
moving a file into a folder. Suppose we have the following file structure.

```
/home
  /my-folder
  my-file
```

Suppose I would like to move the file my-file into the folder /my-folder. The most primitive way to do this
is to run a terminal in the location of /home and use the command

```
>>> mv my-File my-folder/my-file
```

This command calls the move executable (mv) and tells it to move the file my-file to the location
my-folder/my-file. This is synonymous to pull out the move-file form and filling in the right parameters.

```
Move File Form:
  file: _____
  to: _______
```

If you are in the file explorer on your computer, to if you want to move my-file into /my-folder what you
would probably do is drag the icon that represents my-file onto to icon that represents /my-folder. However,
since we know that under the hood the file is actually moved by the command mentioned earlier.

I think that we can interpret it as follows, when you press on and start to drag the my-file icon, then
computer program opens the move-file form and fills in the file: field with the string my-file, then when
you move it over the hover over the /my-folder icon it fills in the to: field with the string
/my-folder/my-file and finally when you drop the icon it presses the go button.

Similarly if you want to create the same affect but you are on a mobile phone,  you would probably open the
mobile file explorer and navigate to the location of my-file, then select my-file and press the move button
which causes the program to pull out the correct form and fill in the file: field with my-file. The program
then places you back to the home directory, and then you navigate to the folder you want to move the file
into and press on it, and the program fills in the ...
