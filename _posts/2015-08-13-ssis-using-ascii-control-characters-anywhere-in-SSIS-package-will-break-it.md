---
title: 'SSIS: Using ASCII control characters anywhere in a SSIS package will break it'
layout: post
---
I noticed this oddity when I was reading a CSV file using SSIS. Sometimes the source file would have a the ASCII SUB control character in the first column on the last line of the file (see screenshot below):

<!--break-->

![ASCII SUB control character from .csv file](ssis-control-character-breaks-things.png)

My first attempt to remove the line from Data Flow pipeline was to add a Conditional Split transformation to the Data Flow task. There, I checked if the first column was equal to the character, as seen in the screenshot below. 

![Filtering away rows with a ASCII SUB control character in the first column](ssis-control-character-breaks-things-conditional-split-fail.png)

The editor did not complain, however, when I tried to execute the task, nothing happened. After few frustrating minutes I tried to restart Visual Studio, but when I reopened my package and looked in my Data Flow task, it was completely empty. 

My guess is that adding any [ASCII control character](http://www.robelle.com/smugbook/ascii.html), i.e. non-printing character, to somewhere in a SSIS package will caused the XML engine in Visual Studio to choke, and the result was that nothing was saved to my packaged .dtsx file.

So how do we use ASCII control characters in SSIS. The right (and now obvious) approach is to use the [CODEPOINT](https://msdn.microsoft.com/en-us/library/ms137696.aspx) and lookup number of the ASCII control character we want to match with in an [ASCII table](http://www.robelle.com/smugbook/ascii.html), e.g.:

![Filtering away rows with a ASCII SUB control character in the first column using the CODEPOINT function](ssis-control-character-breaks-things-conditional-split-works.png)

Hope this helps!