---
layout: post
title: TYPO3 table wizard limitations in TYPO3 11.5+
date: '2023-03-26T18:16:00.000+02:00'
author: Torben Hansen
tags:
- TYPO3
- table wizard
- performance
modified_time: '2023-03-30T11:18:00.000+02:00'
permalink: /2023/03/2023-03-26-typo3-table-wizard-limitations.html
---

I recently updated a TYPO3 website for one of my clients from TYPO3 10.4 to 11.5. The whole update went pretty
smooth and the client was happy with the new version except one thing. The table wizard in TYPO3 11.5 
(TCA `renderType='textTable'`) was causing some problems, as it now was always rendered. 

The table wizard is really helpful if you want to create a table directly in TYPO3. You can add columns and rows 
as shown on the screenshot below.

![TYPO3 table wizard](/assets/images/2023-03-26/typo3-rendertype-textTable.png)

My client however used to copy data from a CSV file and paste it to the `bodytext` field of the table content element.
Usually, the pasted CSV consisted of **several thousand lines** of data, so the TYPO3 table wizard had to render many 
thousands of input fields in the table wizard, which was **really slow**. ~~But that was not the only limitation I had to 
deal with, since the data was not saved to the TYPO3 database. I quickly realized, that the PHP variable 
`max_input_vars` was causing this problem, since it was set to `1500` - the minimal recommended value if you use TYPO3.
Since the table wizard created many thousand input fields for the huge tables, PHP refused to accept all data in the 
POST request, which resulted in the described problem that not all data was saved.~~ Update 30.03.2023 - the problem
with the `max_input_vars` has been [fixed](https://forge.typo3.org/issues/100354) in TYPO3 11.5 and 12.

The solution for this problem was easy. I just disabled the table wizard using a TCA override as shown below:

```
unset($GLOBALS['TCA']['tt_content']['types']['table']['columnsOverrides']);
```

This disables the TYPO3 table wizard for the table content element and my client could again easily copy and paste
CSV data into the `bodytext` field.