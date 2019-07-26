---
title: Announcing Regex Nodes, an online Editor for Regular Expressions
# jekyll-seo-tag
description:  "Visualizing and editing regular expressions in the web"
#image:        "http://placehold.it/400x200" # why nothing visible?
author:       "johannesvollmer"
---

Hey! In this post, I am going to present to you 
what I have built to simplify creating regular expressions for JavaScript:
[Regex Nodes](https://johannesvollmer.github.io/regex-nodes/).

*Warning: The presented project is still work in progress and some features
might be missing or may not work as intended. See the 
[github project](https://github.com/johannesvollmer/regex-nodes)
for more information on which features are planned.*


# What are Regular Expressions?

Regular expressions are used to find certain 
patterns inside a given text. For example, it can be used to extract
all images from an html page. The way to use regular expressions in
JavaScript is to just type a textual representation of the pattern
inbetween two slashes:

```JavaScript
const regex = /<img.*?>/gim
htmlSourceCode.search(regex)
```

This code will search the `htmlSourceCode` for any occurrence of 
`<img` followed by the lowest repetition (`*?`)  of any character (`.`), 
followed by a `>`. After the slash, we tell JavaScript to be 
case insensitive and to look for multiple matches, not just for the first.

# The Approach of the Editor

The editor visualizes all components of a regex with a separate "Node".
Those atomic components can be composed by connecting the properties of the nodes.
That way, you can compose complex regular expressions 
without having to deal with overly complex text lines.

The following image shows the editor with the aforementioned regular expression.

[
    ![Filter Image Tags in the "Regex Nodes" Editor]({{ site.baseurl }}/img/regex-nodes/image-tags.png)
]({{ site.baseurl }}/img/regex-nodes/image-tags.png)

In the center, the white rectangles describe the hierarchy of the expression.
The rightmost node is the top-level operator, which combine the atomic
nodes on the left hand side to a single regular expression.

To construct and play around with that node setup yourself,
simply visit [the online editor](https://johannesvollmer.github.io/regex-nodes/), 
paste the expression `/<img.*?>/gim` into the "Add Nodes" text box,
and then click the first suggestion. To see how the regex works,
you will also have to edit the example text 
by clicking the button at the top right and paste some HTML, as 
the default example text does not contain any image tags.
Be aware that padding spaces in the regular expression 
will not be ignored and may lead to unexpected results.

# Why Nodes?

Nodes are more verbose than a textual regex, 
but enable you to understand even the most complex expressions.
Also, the design of the editor will naturally prevent you 
from accidentally producing an incorrect regular expression. 

This really sets regex nodes apart from other regex tools, where
the user must first come up with a regular expression
that will be checked afterwards.


# Basic Editor Functionality

As I have not yet done any user experience research 
on the interface design, I will explain the basic functionality here:

1. __Select a node__ by clicking with your left mouse button. 
   Selecting a node will display its regular expression at the bottom. 
   You can lock your selection to the currently selected node by toggling 
   the padlock at the bottom left. 
1. __Move a node__ by dragging with your left mouse button
1. __Connect or disconnect properties__ by dragging with your right mouse button
1. __Move the view__ by scrolling, or by dragging with your middle mouse button.
1. __Add a node__ by clicking into the "Add Nodes" text field 
   and choosing the desired node type. You can also paste a regular expression here.
1. __Edit the example text__ which is displayed in the background 
   by clicking on the button at the top right.
1. __Undo and redo__ your actions with the arrows at the top.
1. __Duplicate or delete a node__ by selecting it and then clicking the 
   respective button at the top of the node which just appeared.
1. Select and CTRL-C copy the resulting regular expression 
   and paste it into your JavaScript code. 
   If you like this project, 
   link this editor in a comment right next to the regular expression.


# Troubleshooting

1. Problem: The editor does not work in realtime 
   because clicking a node stops the editor for a significant amount of time. 

   Try: Reduce the length of the example text. 
   A shorter text will need less time to highlight all matches
   when a node is selected.

   Alternatively, lower the "Example Match Limit" while editing the example text.
   This will prevent the editor from highlighting more than X matches.

# Finally

Thanks for reading! I hope you will find this editor useful 
for your own regular expressions. 


Keep in mind that this editor is still work-in-progress, and I cannot make
any guarantees concerning the reliability of the editor. 
To see what features are planned, see the [github project](https://github.com/johannesvollmer/regex-nodes).
Don't hesitate to post an issue on github 
if something comes to your mind regarding the regex editor.

Have fun!
