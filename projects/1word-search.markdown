---
layout: post
isProject: true

title: Word Search
description: Solving word search with the Boyer–Moore string-search algorithm.

tags: Java
---

**Github Page:** [https://github.com/saahbha/word-search-java](https://github.com/saahbha/word-search-java)

In this article, I discuss how to write a program to solve word search puzzles with the Boyer–Moore string-search algorithm.
Above is a link for the github page for my implementation of these ideas written in Java. Come back later for a live, hosted demo written in React.js!

![](\assets\projects\word-search\ex-puzzle.png)

# Introduction

When word search puzzles were first published in newspapers in the late 60s, they took to schools by storm. Now everyone has memories of playing with word search puzzles when they were young. In fact, one thing I remember from elementary school is that it was almost a competition for glory to see who can do them faster. Maybe you still do word search puzzles every once in a while. Good on you! Frequently doing puzzles like word search among others is a great way to strengthen your vocabulary and problem solving skills.
I think these puzzles are a great way to demonstrate the power of algorithmic problem solving. Everyone has methods and strategies to solve word search. But in this article, I'll describe a set of concise rules that I used to write a Java program to solve word search puzzles quickly.

# Breaking it down

The common strategy is to traverse the word search grid as if we were reading a book: left to right, up to down. When we find a letter that matches the first letter of a word we are looking for, we search in all directions around the letter: up, down, left, right, and diagonally. The method I describe, is somewhat in reverse. We traverse the gid in each of the directions and see we find the word with the Boyer-Moore algorithm.

#### To put it in 3 main steps:

- Get all possible strings in each direction
- For each direction, try to find the word in each of the strings in the direction with the Boyer-Moore string search algorithm.
- If we find the word, output the coordinate of the first letter of the word where it shows up in the word search puzzle.

# Searching the puzzle

![](\assets\projects\word-search\ex-directions.png)
The Boyer-Moore algorithm takes a string to search in and a word to find in the given string, and outputs the index at which the word starts in the string. So to apply this to word search, we need to generate a string for each direction in the puzzle and be able to convert string coordinates to word search grid coordinates. It's fairly easy for us to be able to just look at the puzzle and imagine it without dealing in these numbers, but a program needs to be able traverse the puzzle in terms of indices and coordinates.

To get all the strings that can be made by reading in a given direction, we need to traverse the grid in that direction and put it in an array or list made for that direction. To understand how to do this, I made the following figures for each direction. I colored the strings that can be made by reading in each direction to make it easy to understand where the coordinates for that string come from. The first two columns in the tables on the right after the color describe the array of strings for a direction. The Value is the string, and the Index is where the string should show up in the array. The column of Word Search Coordinates describe where each letter in the string shows up in the grid. The column of String Coordinates describe where each letter in the string shows up in the string array.

- The coordinates for a letter in the word search grid are `(row, column)`
- The coordinates for a letter in the string are `(dirIndex, strIndex)` where `strIndex` is the index of the letter in the String and `dirIndex` is the index of the String in the Direction array.
- To go from the coordinates for a letter in the string to the letters coordinates in the array, we just use equations of the form: `(row, column) = (f(dirIndex, strIndex, dim), g(dirIndex, strIndex, dim))` where `dim` is the dimension of the puzzle grid, and `f(...)` and `g(...)` are functions that turn their inputs into word search coordinate components corrisponding to `row` and `column`.

### Row Strings

![](\assets\projects\word-search\ex-row-strings.png)
For the Array of Row Strings, it's really simple. Iterate over each row, building the string for that row by iterating over each column in the row. We see that String Coordinates are exactly the same as the Word Search Coordinates, so `(row, column) = (dirIndex, strIndex)`.

In Java, the code to do this looks like this:

```Java
private static String[] getRows(final char[][] puzzleGrid) {
    int dim = puzzleGrid[0].length;
    String[] rows = new String[dim];
    //Get Row Strings from the puzzleGrid
    //iterate over rows
    for (int m = 0; m < dim; m++) {
        rows[m] = "";
        //iterate over cols
        for (int n = 0; n < dim; n++) {
            rows[m] += puzzleGrid[m][n];
        }
    }
    return rows;
}
```

### Column Strings

![](\assets\projects\word-search\ex-col-strings.png)
For the Array of Column Strings, it's slightly less simple. It's the same as the array of row strings, but reversed in a way. We can iterate over each column, building the string for that column by iterating over each row in the column. We see that Word Search Coordinates in terms of String Coordinates are `(row, column) = (strIndex, dirIndex)`.

In Java, the code to do this looks like this:

```Java
private static String[] getVertical(final char[][] puzzleGrid) {
    int dim = puzzleGrid[0].length;
    String[] columns = new String[dim];
    //Get Column Strings from the puzzleGrid
    //iterate over cols
    for (int n = 0; n < dim; n++) {
        columns[n] = "";
        //iterate over rows
        for (int m = 0; m < dim; m++) {
            columns[n] += puzzleGrid[m][n];
        }
    }
    return columns;
}
```

### Ascending Diagonal Strings

![](\assets\projects\word-search\ex-adiag-strings.png)
This is where things get more complex. There are more strings in these directions and the starting and ending sequence is different for the last half. We see that Word Search Coordinates in terms of String Coordinates for `strIndex` less than `dim` are `(row, column) = (dirIndex - strIndex, strIndex)`. When the `strIndex` is greater than or equal to `dim`, the coordinates are `(row, column) = (dim - 1 - strIndex, dirIndex - dim + 1 + strIndex)`.

We see that when building each string, we always start at `(starting row, 0)` and end at `(0, starting row)` in word search coordinates, atleast until after starting row index `dim-1`.

- So until this point, we build each string by iterating over each row and for each row iterate over each column starting at column 0 such that `col <= startRow`, the coordinates for the letter in each iteration are then `(startRow - col, col)`.

After row index `dim-1`, we see that strings always start at `(dim - 1, starting column)` and end at `(starting column, dim - 1)`.

- Here we build each string by iterating each column and for each column iterate over each row in reverse starting at column number `dim-1` such that `startCol <= row`, the coordinates for the letter in each iteration are then `(row, startCol + (dim - 1) - row)`.

In Java, the code to do this looks like this:

```Java
private static String[] getAscendingDiagonals(final char[][] puzzleGrid) {
    int dim = puzzleGrid[0].length;
    String[] ascendingDiagonal = new String[dim*2 - 1];
    for (int startRow = 0; startRow < dim; startRow++) {
        ascendingDiagonal[startRow] = "";
        for (int col = 0; col <= startRow; col++) {
            ascendingDiagonal[startRow] += puzzleGrid[startRow - col][col];
        }
    }
    for (int startCol = 1; startCol < dim; startCol++) {
        ascendingDiagonal[dim + startCol - 1] = "";
        for (int row = dim - 1; row >= startCol; row--) {
            ascendingDiagonal[dim - 1 + startCol] += puzzleGrid[row][startCol + (dim - 1) - row];
        }
    }

    return ascendingDiagonal;
}
```

### Descending Diagonal Strings

![](\assets\projects\word-search\ex-ddiag-strings.png)
Here see that we see that Word Search Coordinates in terms of String Coordinates are for strIndex less than dim are `(row, column) = (strIndex, dim - 1 - direction array index + strIndex)`. When the strIndex is greater than or equal to `dim`, the coordinates are `(row, column) = (direction array index - dim + 1 + strIndex, strIndex)`.

When building each string, we always start at `(0, starting column)`, where the starting column goes in reverse, and end at `(dim - 1 - starting column, dim)`, in word search coordinates, atleast until after starting column index 0.

- So until this point, we build each string by iterating over each column in reverse, and for each column iterate over each row such that `row <= (dim - 1 - startCol)`, the coordinates for the letter are then `(row, startCol + row)`.

After starting column index 0, strings always start at `(starting row, 0)` and end at `(dim - 1, dim - 1 - starting row)`.

- Here we build each string by iterating each row starting at row index 1 and for each row iterate over each column such that `col <= dim - 1 - startRow`, the coordinates for the letter in each iteration are then `(startRow + col, col)`.

In Java, the code to do this looks like this:

```Java
private static String[] getDescendingDiagonals(final char[][] puzzleGrid) {
    int dim = puzzleGrid[0].length;
    String[] descendingDiagonal = new String[dim*2 - 1];
    for (int startCol = dim - 1; startCol >= 0; startCol--) {
        int arrIndex = dim - 1 - startCol;
        descendingDiagonal[arrIndex] = "";
        for (int row = 0; row <= arrIndex; row++) {
            descendingDiagonal[arrIndex] += puzzleGrid[row][startCol + row];
        }
    }
    for (int startRow = 1; startRow < dim; startRow++) {
        int arrIndex = dim - 1 + startRow;
        descendingDiagonal[arrIndex] = "";
        for (int col = 0; col <= dim - 1 - startRow; col++) {
            descendingDiagonal[arrIndex] += puzzleGrid[startRow + col][col];
        }
    }

    return descendingDiagonal;
}
```

# Boyer-Moore

The main idea of Boyer-Moore is to line up the word in the string of letters we are looking in. The catch is that we start looking from the end of the word. The steps for the algorithm is below:

- Start at the beginning of the string and search through a selection that is the size of the word we are looking for. Compare letters in the word to letters in the string starting from the end of the selection and the end of the word, making our way to the start of the word.
- If we find a mismatch, check if the mismatched letter in the string shows up in the word.
  - If the letter does not show up in the word, skip past the end of the selection because the word can't possibly be in it.
  - If it does show up in the word, line up the word with the last place that this mismatched letter occurs in the word and check again.

To show an example of how this works, let's say we want to find the word "seashore" in the string "she_sells_seashells_by_the_seashore". Here is what Boyer-Moore does:

![](\assets\projects\word-search\ex-boyer-moore-string.png)
![](\assets\projects\word-search\ex-boyer-moore-LOF.png)
![](\assets\projects\word-search\ex-boyer-moore-table.png)

- The rows labeled 'shift' describe the current selection of the algorithm is searching in.
- Letters colored grey are letters that were not compared.
- Letters colored green are letters that were compared and found to be matches.
- Letters colored red are letters that were compared and found to be mismatches.
- Letters colored orange are letters that occurred at a mismatch in the process and found to also be in the word.

Once again, the Boyer-Moore algorithm starts searching from the end of the word. If a mismatch occurs, we need to shift our searching selection right. We see that there are 6 mismatches, thus 6 shifts were needed.

- At first, we find a mismatch immediately at the last 'e' in the word. The mismatch letter in the string is 'L' and this doesn't show up in the word so we shift, skipping over the mismatch since we already know that the word doesn't contain it. This is shift 1. We don't have to check the rest of the selection because the size of it is the size of the word, and if there is a single mismatch then the word can not possibly be in the selection.
- After shift 1, we find a mismatch at the last 'r' in the word. The mismatch letter in the string is 'h' which does also occur in the word, so we make a shift lining up 'h' in the string with the last place 'h' occurs in the string. This is shift 2.
- After shift 2, we again find a mismatch immediately at the last 'e' in the word. Again the mismatch is at 'L' and this doesn't show up in the word. So we shift, skipping over the mismatch. This is shift 3.
- After 4 more shifts, we finally find "seashore" at the end of the string.

A variation of the pseudocode for the Boyer-Moore Algorithm is as follows.

```Java
//T: A string to search in
//P: A word to find
//∑: The set of all letters in T and P
Algorithm BoyerMoore(T, P, ∑)
	L = lastOccurenceFunction(P, ∑)
	m = P.length    //length of the word
        i = m - 1       //increments over the string
        j = m - 1       //increments over the word
	Repeat
		If T[i] == P[j] //If matched
			If j = 0 //finished searching the section with no mismatches
				Return i
			Else //not done searching
				i = i - 1
				j = j - 1
		Else //Mismatch occurs
			last = L[T[i]] //find the last occurrence of the mismatched letter in P
			i = i + m - min(j, 1 + last) //shift the search area of T
			j = m - 1 //start searching from the end of P again
	Until i > n-1
    Return -1 // no match found
```
