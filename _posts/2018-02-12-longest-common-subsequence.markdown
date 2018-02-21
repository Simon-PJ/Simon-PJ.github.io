---
layout: post
title:  "Longest Common Subsequence"
date:   2018-02-21 19:52:00 +0000
categories: jekyll update
---

# Introduction

The longest common subsequence problem is exactly as the name suggests: the problem of finding the longest common
subsequence in a set of sequences. The solution to the problem can be used to determine the differences
between two pieces of text - each piece of text is a sequence. There are libraries available to do this work for us,
but I recently needed a bit more flexibility than they could provide and so investigated how I could apply this in
my own work. There are a number of sources already available that explain algorithms for solving this problem, but I
feel they could be simplified. This is my attempt at doing so.

# Worked example

The easiest way to understand the algorithm is to work through an example. Here we will be comparing two sentences:

> This is some text to compare
>
> This is some more text

If we display deleted words in red and new words in blue we would expect a differential to produce the following:

> This is some <span style="color:blue">more</span> text <span style="color:red">~~to compare~~</span>

To get this result we need to build a matrix. The column headers for this matrix should be each of
the before words in sequence. The row headers should be each of the after words in sequence. The
first column and row should be an origin item. Setting this up gives:

![Matrix 1](/assets/lcs-1.png)

Next up, the first column and row should be filled with 0s like so:

![Matrix 2](/assets/lcs-2.png)

Now it gets a little more complicated as we fill the rest of the table with numbers. There are two
rules that are checked in sequence to determine the value of each cell. These are:

1. If the column header and row header match, take the value directly upper left and add one.
2. Else take the greater value of the cell above and the cell to the left.

This gives:

![Matrix 1](/assets/lcs-3.png)

The above table can now be used to work out the differences between the two sentences. Again, there
are some rules to follow. When applying these rules we start at the bottommost right cell and we act
on the first that matches for each cell.

1. If the column and row headers match, the word is unchanged. Move to the cell up and to the left.
2. If the column and headers do not match, and the cell above is greater than or equal to the cell to the left, the row header word is inserted. Move to the cell above.
3. Else the column header word has been deleted. Move to the cell to the left

The process continues until the cell is (0,0). Doing this for our example produces the following path:

![Matrix 1](/assets/lcs-4.png)

Now we can work through this path and determine the diff based off the earlier rules:

- Position (6,5): compare != text. 4 > 3 therefore move to (5,5). Moving left therefore 'compare' has been deleted

> <span style="color:red">~~compare~~</span>

- Position (5,5): to != text. 4 > 3 therefore move to (4, 5). Moving left therefore 'to' has been deleted.

> <span style="color:red">~~to compare~~</span>

- Position (4,5): text = text thefore move to (3,4). Text is the same therefore 'text' is unchanged.

> text <span style="color:red">~~to compare~~</span>

- Position (3,4): some != more. 3 > 2 therefore move to (3,3). Moving up therefore 'more' has been inserted.

> <span style="color:blue">more</span> text <span style="color:red">~~to compare~~</span>

- Position (3,3): some = some therefore move to (2,2). Text is the same therefore 'some' is unchanged.

> some <span style="color:blue">more</span> text <span style="color:red">~~to compare~~</span>

- Position (2,2): is = is therefore move to (1,1). Text is the same therefore 'is' is unchanged.

> is some <span style="color:blue">more</span> text <span style="color:red">~~to compare~~</span>

- Position (1,1): is = is therefore move to (0,0). Text is the same therefore 'This' is unchanged.

> This is some <span style="color:blue">more</span> text <span style="color:red">~~to compare~~</span>

And there we have the result of our diff. Looking back to the start of the example, this result matched the expected
result of the differences between the two sentences which is a relief!