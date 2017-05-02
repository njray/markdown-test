#Markdown information

#Mike’s cheat sheet 


By Mike Wasson– I’m probably forgetting some things 

##General Markdown


Use GitHub flavored markdown.
<https://guides.github.com/features/mastering-markdown/>

##Metadata


Put this at the top of each file:

\---

title: \<article title\>

description: \<description for SEO\>

author: \<your GitHub user name\>

ms.date: \<mm/dd/yyyy\>

\---

##Links

Link to **docs in our repo**: Use a relative link to the MD file.
```
../foo/bar.md
```

When linking to a file in the same folder, we prefer using the “initial dot” notation:

```
               ./bar.md
```

Link to **other docs on docs.microsoft.com** (including Azure product docs, ASP.NET, SQL Server, .Net): Use the URL path segment (everything after the hostname)

```
               /azure/virtual-machines/windows/create-availability-set
```

Links to **all other websites** (including MSDN, technet): Use the full URL.

For Microsoft sites, strip the locale string (e.g., “en-us”).

-   But test the non-locale version, some

##Notes

We have a special syntax for formatted notes.

\> [!NOTE]

\> This is a note

There is also [!WARNING], [!TIP], and [!IMPORTANT]

These render in a shaded box. They are actually kind of jarring visually, so use them sparingly. “Warning” is useful for something that’s a security concern, e.g.:

\> [!WARNING]

\> Change the example password in the configuration file, before you deploy.

# Writage: Markdown plug-in for Word

By Tycen Hopkins

Get [Writage](http://www.writage.com/)


In the past I’ve used it to convert big complicated docs like this in 15-30 minutes:

<https://docs.microsoft.com/en-us/azure/architecture/aws-professional/services>

Once it’s installed you can save as markdown files from word docs. Once it’s spit out there’s clean-up tasks that still need to be done after export:
 

1. Convert heading formats from the underline formats:

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

2. Images get generated with random guids as filenames and without alt text, so:

 

![](media/64e97219208353a9410e7ca9c1d5b614.png)

becomes something like :

![Administer emails settings](admin-instructions/admin-email.png)

3. Code samples formatting needs to be fixed. Copy the code sample from the word
doc and paste to the markdown code, surrounding it with the special code block
characters (note these are not just single quotes, but the fancy curly quote
\`\`\`):

 

\`\`\`

myFunction(){

                x=x;

}

\`\`\`

4. Adding the YML metadata to the file.

 

You can also open markdown files in word, although you’ll lose some of the
markdown formatting features if you do so.

 

\-Tycen
