# How to Read a CSV File using a Different Delimiter

## Introduction

In this guide, I will show how to read a CSV file using a different **column** delimiter than the presets available on the OIC FTP adapter. These are the space, comma, semicolon, tab, and pipe characters ` ,;	|`. Each (column) field may be optionally enclosed by these characters: double quote, single quote, parentheses, square bracket, curly brace, angle bracket, or pipe characters `"'()[]{}<>|`. Finally, each row is terminated by the END_OF_LINE character, newline, carriage return, or both `${eol}` `\n` `\r` `\r\n`.

Normally, a CSV file will use commas, double quotes, and `${eol}` as its column delimiter, field separator, and row delimiter: `person.csv`

```
Fname,Lname,email,phone,BID
Alice,Sundayala,alice.sundayala@email.com,3004005000,1
Brent,Markleson,brent.markleson@email.com,1002003000,1
Craig,Stromdell,craig.stromdell@email.com,3101234567,4
Dylan,Renderman,dylan.renderman@email.com,8857713312,2
Ellie,Heartford,ellie.heartford@email.com,3005006000,1
Franz,Guillermo,franz.guillermo@email.com,1311411511,3
Grant,Zapanicca,grant.zapanicca@email.com,3104456667,4
Haley,Riverbend,haley.riverbend@email.com,8851357900,2
```

In this guide, I will be working with the following CSV file: `personhash.csv`

```
Fname#Lname#email#phone#BID
Alice#Sundayala#alice.sundayala@email.com#3004005000#1
Brent#Markleson#brent.markleson@email.com#1002003000#1
Craig#Stromdell#craig.stromdell@email.com#3101234567#4
Dylan#Renderman#dylan.renderman@email.com#8857713312#2
Ellie#Heartford#ellie.heartford@email.com#3005006000#1
Franz#Guillermo#franz.guillermo@email.com#1311411511#3
Grant#Zapanicca#grant.zapanicca@email.com#3104456667#4
Haley#Riverbend#haley.riverbend@email.com#8851357900#2
```

where the column delimiter has been replaced by a hash symbol `#`.

## Auxiliary Information

This guide was written on November 22, 2021. The version of OIC that was used to create these integrations was version 21.4.2.

## Assumptions

This guide assumes the following:
- The CSV file originates from a file server and is accessed via the FTP adapter.
- The file does NOT contain delimiter characters that you will be using to replace the existing delimiter characters. For instance, if you wish to replace `A#B#C#D#E, Euler's Number\r\n` with `A,B,C,D,E, Euler's Number\r\n` the extra comma before ` Euler's` will result in this row of data having six fields instead of five. For a more detailed discussion on how to handle these situations, see [this section](#Advanced-Methods-for-Parsing-Existing-Delimiter-Characters).

## Prerequisites

Please make sure you understand the following concepts prior to reading this guide. The details for performing the following will not be written in this guide.
- How to activate/deactivate integrations
- How to provision and administer your OIC instance
- How to navigate to your OIC homepage
- Where to create connections
- Where to create integrations


## Method 1: Change Entire Row

In this method, I will download the file from the file server, then use the following pseudocode mapping to map each entire row to a new entire row where all old delimiters are replaced by new ones:

```
create-delimited-string(
	create-nodeset-from-delimited-string(
		$qname,
		$some_delimited_string,
		$old_delimiter_character
	), $new_delimiter_character
)
```

Explanation for variables:
- `$qname`: a string of the form `{a}b` or `a:b` which represents a namespace. The first form generally looks something like `{http://test.com/xmlns}value` and the second one could be `namespace:value`.
- `$some_delimited_string`: a string that contains row data. This could be something like `A#B#C#D#E`.
- `$old_delimiter_character`: a single character that separates all the fields in `$some_delimited_string`. In this example it would be `#`.
- `$new_delimiter_character`: a single character to replace `$old_delimiter_character`. Maybe I want my row data to look like `A,B,C,D,E`, meaning this character should be `,`.

### Step 1: Create Integration

### Step 2: Download File

### Step 3: Upload File

### Step 4: Define Mapping

### Step 5: Test, 

## Method 2: Map Parsed Row to Individual Target Elements

s

## Advanced Methods for Parsing Existing Delimiter Characters

## Closing

ssdf





