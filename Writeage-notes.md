# Writage: Markdown plug-in for Word

By Tycen Hopkins

Get [Writage](http://www.writage.com/)


In the past I’ve used it to convert big complicated docs like this in 15-30 minutes:

<https://docs.microsoft.com/en-us/azure/architecture/aws-professional/services>

Once it’s installed you can save as markdown (.md) files from Word docs. Once it’s spit out, there’s clean-up tasks that still need to be done after export:
 

## 1. Convert heading formats from the underline formats:

>    

>   My Heading 1

>   ===========

>    

>   My Heading 2

>   \------------------

>    

>   To the \# format:

>   \# My Heading 1

>   \#\# My Heading 2

## 2. Images get generated with random guids as filenames and without alt text, so:

![](media/64e97219208353a9410e7ca9c1d5b614.png)

becomes something like :

![Administer emails settings](admin-instructions/admin-email.png)

## 3. Code samples formatting needs to be fixed. 

Copy the code sample from the Word doc and paste to the markdown code, surrounding it with the special code block
characters (note these are not just single quotes, but the fancy curly quote \`\`\`):

\`\`\`

myFunction(){

                x=x;

}

\`\`\`

## 4. Adding the YML metadata to the file.

You can also open markdown files in word, although you’ll lose some of the
markdown formatting features if you do so.

 

\-Tycen
