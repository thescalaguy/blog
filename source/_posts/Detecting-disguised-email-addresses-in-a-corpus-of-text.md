---
title: Detecting disguised email addresses in a corpus of text
tags:
  - python
date: 2024-08-03 11:38:36
---


I'd recently written about [an experimental library to detect PII.](/2024/08/02/An-experiemental-library-to-detect-PII/) When discussing it with an acquaintance of mine, I was told that PII can also be disguised. For example, a corpus of text like a review or a comment can contain email address in the form "johndoeatgmaildotcom". This led me to update the library so that emails like these can also be flagged. In a nutshell, I had to update the regex which was used to find the email.  

## Example

This is best explained with a few examples. In all of the examples, we begin with a proper email and disguise it one step at a time.

{% code lang:python %}
column = Column(name="comment")
detector = ColumnValueRegexDetector()

assert detector.detect(column=column, sample="john.doe@provider.com") == Email()
assert detector.detect(column=column, sample="johndotdoe@provider.com") == Email()
assert detector.detect(column=column, sample="johndotdoeatproviderdotcom") == Email()
assert detector.detect(column=column, sample="john.doe@provider.co.uk") == Email()
assert detector.detect(column=column, sample="johndotdoeatproviderdotcodotuk") == Email()
assert detector.detect(column=column, sample="myemailis:john.doeatproviderdotcom") == Email()
{% endcode %}  

All of these assertions pass and the regex detector is able to flag all of these examples as email.