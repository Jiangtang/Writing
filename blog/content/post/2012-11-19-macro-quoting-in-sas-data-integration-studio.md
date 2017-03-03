---
id: 792
title: Macro Quoting in SAS Data Integration Studio
date: 2012-11-19T21:35:04+00:00
author: Jiangtang Hu
layout: post
guid: http://www.jiangtanghu.com/blog/?p=792
permalink: /2012/11/19/macro-quoting-in-sas-data-integration-studio/
categories:
  - SAS
tags:
  - Macro
  - Macro Quoting
  - SAS
  - SAS Data Integration Studio
---
<font size="1">You can run the following piece of codes successfully in these 3 SAS programming environments:</font>

  * <font size="1">BASE SAS </font>
  * <font size="1">Enterprise Guide: create a new “File-New-Program” </font>
  * <font size="1">SAS Data Integration Studio: create a new “Tool-Code Editor” </font>

> <font size="1" face="Courier New">%let species="Setosa" "Versicolor";</font>
> 
> <font size="1"><font face="Courier New">data a; <br />&#160;&#160;&#160; set sashelp.iris; <br />&#160;&#160;&#160; where species in (&species); <br />run;</font> </font>
> 
> <font size="1" face="Courier New"></font>

<font size="1">Then create a <font color="#00ff00">Transformation</font> in SAS Data Integration Studio (DIS for short; I use version 4.3 in a Win 7 machine) using the codes above as source code (remember deleting first line) and create a simple <font color="#00ff00">Prompt</font> to assign the macro variable &species with default values as <em>"Setosa" "Versicolor"</em>:</font>

[<img style="background-image: none; border-right-width: 0px; margin: 3px auto 5px; padding-left: 0px; padding-right: 0px; display: block; float: none; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="SAS_DIS_Transformation" border="0" alt="SAS_DIS_Transformation" src="http://www.jiangtanghu.com/blog/wp-content/uploads/2012/11/SAS_DIS_Transformation_thumb.png" width="519" height="475" />](http://www.jiangtanghu.com/blog/wp-content/uploads/2012/11/SAS_DIS_Transformation.png)

<font size="1">Drag this <font color="#00ff00">Transformation</font> in a single node <font color="#00ff00">job</font> and run, then you will get such errors:</font>

> <font face="Courier New">127&#160;&#160;&#160;&#160;&#160;&#160;&#160; data a; <br />128&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; set sashelp.iris; <br />129&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; where species in (&species); <br /><font color="#ff0000">NOTE: Line generated by the macro variable "SPECIES". <br />129&#160;&#160;&#160;&#160;&#160;&#160;&#160; "Setosa", "Versicolor"</font> <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; _ <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 22 <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; _ <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 76 <br /><font color="#ff0000">ERROR</font>: Syntax error while parsing WHERE clause. <br /><font color="#ff0000">ERROR</font> 22-322: Syntax error, expecting one of the following: a quoted string, a numeric constant, a datetime constant, <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; a missing value, -.&#160; </font>
> 
> <font face="Courier New"><font color="#ff0000">ERROR</font> 76-322: Syntax error, statement will be ignored.</font>
> 
> <font face="Courier New">130&#160;&#160;&#160;&#160;&#160;&#160;&#160; run;</font>

<font size="1">So, what happened inside DIS since such codes go well in BASE SAS, SAS EG and the Code Editor in DIS itself? The default value <em>"Setosa" "Versicolor"</em> was assigned to macro variable &species in the DIS <font color="#00ff00">Prompt</font> (see picture above) and you would expect the following effect like what I wrote in open codes in BASE SAS:</font>

> <font size="1" face="Courier New">%let species="Setosa" "Versicolor";</font>

<font size="1">Actually NO. In DIS, this action was translated into such clause:</font>

> <font face="Courier New">%let species = <font color="#ff0000">%nrquote</font>(%"Setosa%", %"Versicolor%");</font>

<font size="1">Here is the point: since there quotation symbols were masked by <font color="#ff0000"><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p0ejcenxk89b36n1xgxyf922ugoa.htm" target="_blank"><font color="#00ff00">%nrquote</font></a> </font>function, they are no longer valid syntax characters (then we got errors!). </font>

<font size="1">The solution: use a </font><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p1k3cotqhvgwk0n105gyfe1cqg9m.htm" target="_blank"><font size="1">%<font color="#00ff00">unquote</font></font></a> <font size="1">function on the macro variable &species to reverse the masking effort by <font color="#ff0000"><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p0ejcenxk89b36n1xgxyf922ugoa.htm" target="_blank"><font color="#00ff00">%nrquote</font></a></font><font color="#cccccc">(note in our open codes example in BASE SAS, both functions were not used):</font></font>

> <font size="1"><font face="Courier New">data a; <br />&#160;&#160;&#160; set sashelp.iris; <br />&#160;&#160;&#160; where species in (<font color="#ff0000">%unquote</font>(&species)); <br />run;</font> </font>

# <font size="1"><font size="1"></font></font>

<font style="font-weight: bold">Notes on Macro Variable Generated by DIS</font></h1> 

> <font face="Courier New">%let species = <font color="#ff0000">%nrquote</font>(%"Setosa%", %"Versicolor%");</font>

<font size="1">1)<font color="#ff0000"><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p0ejcenxk89b36n1xgxyf922ugoa.htm" target="_blank"><font color="#00ff00">%nrquote</font></a> </font>function used while it is dated for a long time… <font color="#ff0000"><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p0ejcenxk89b36n1xgxyf922ugoa.htm" target="_blank"><font color="#00ff00">%nrquote</font></a> </font>and </font><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p1780jrqrtwtw7n16x83peo2zpxr.htm" target="_blank"><font color="#00ff00" size="1">%quote</font></a> <font size="1">were replaced by </font><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p06cx7fegzmzpen1m9991yljxiav.htm" target="_blank"><font size="1"><font color="#00ff00">%BQUOTE</font> and <font color="#00ff00">%NRBQUOTE</font> Functions</font></a><font size="1">.</font>

<font size="1">2)All quotation symbols(“) were preceded by a percent sign (%). That’s <font size="1"><font color="#ff0000"><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p0ejcenxk89b36n1xgxyf922ugoa.htm" target="_blank"><font color="#00ff00">%nrquote</font></a> </font>and </font><a href="http://support.sas.com/documentation/cdl/en/mcrolref/62978/HTML/default/viewer.htm#p1780jrqrtwtw7n16x83peo2zpxr.htm" target="_blank"><font color="#00ff00" size="1">%quote</font></a><font size="1"> needed and why they are outdated any more.</font></font>