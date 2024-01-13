---
layout: post
title: "Filtering a HTML table with just Javascript"
tags:
  - html
  - js
  - javascript
---

## Overview

I enjoy creating simple, functional solutions. The code below minimizes backend calls for generating a new table by offering basic search functionality.

Users can filter rows based on specific column content in a case-insensitive manner.

## Example
<script>
  function tableSearch(inputID, tableID, colStart, colEnd) {
    // Declare variables
    let input, filter, table, tr, td, i, j, txtValue;
    // Find the input elemment
    input = document.getElementById(inputID);
    // Find the table
    table = document.getElementById(tableID);
    // Convert the current value of the input to UPPERCASE
    filter = input.value.toUpperCase();
    // Get all of the rows (trs) from the table
    tr = table.getElementsByTagName("tr");
    // Loop through all table rows
    for (i = 1; i < tr.length; i++) {
      // Get all td elements in the current tr
      var tds = tr[i].getElementsByTagName("td");
      // Flag to check if there is a match in any of the td elements
      var hasMatch = false;

      // Loop through the column range that is provided by colStart and colEnd 
      for (j = colStart; j <= colEnd; j++) {
        td = tds[j];
        if (td) {
          // Check if the filter is found in any of the td elements
          txtValue = td.textContent || td.innerText;
          if (txtValue.toUpperCase().indexOf(filter) > -1) {
            hasMatch = true;
            // Exit the loop if a match is found in any td element
            break; 
          }
        }
      }

      // Set the display property based on whether there is a match
      tr[i].style.display = hasMatch ? "" : "none";
    }
  }
</script>
<input id="clientsSearch" type="search" onkeyup="tableSearch('clientsSearch', 'clientsTable', 1, 3)" />
<table class="table" id="clientsTable">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Email</th>
      <th>Phone</th>
    </tr>
  </thead>
  <tbody>
  <tr>
    <td>1</td>
    <td>John Doe</td>
    <td>john.doe@example.com</td>
    <td>(123) 456-7890</td>
  </tr>
  <tr>
    <td>2</td>
    <td>Jane Smith</td>
    <td>jane.smith@example.com</td>
    <td>(234) 567-8901</td>
  </tr>
  <tr>
    <td>3</td>
    <td>Mike Johnson</td>
    <td>mike.johnson@example.com</td>
    <td>(345) 678-9012</td>
  </tr>
  <tr>
    <td>4</td>
    <td>Alice Williams</td>
    <td>alice.williams@example.com</td>
    <td>(456) 789-0123</td>
  </tr>
  <tr>
    <td>5</td>
    <td>Bob Brown</td>
    <td>bob.brown@example.com</td>
    <td>(567) 901-2345</td>
  </tr>
  </tbody>
</table>

## Code
Use:
- `tableID` for the table to filter 
- `inputID` for the search box
- `colStart` for the column to start searching in (Remember the index starts at 0) 
- `colEnd` for the column to finish searching in

```html
<script>
  function tableSearch(inputID, tableID, colStart, colEnd) {
    // Declare variables
    let input, filter, table, tr, td, i, j, txtValue;
    // Find the input elemment
    input = document.getElementById(inputID);
    // Find the table
    table = document.getElementById(tableID);
    // Convert the current value of the input to UPPERCASE
    filter = input.value.toUpperCase();
    // Get all of the rows (trs) from the table
    tr = table.getElementsByTagName("tr");
    // Loop through all table rows
    for (i = 1; i < tr.length; i++) {
      // Get all td elements in the current tr
      var tds = tr[i].getElementsByTagName("td");
      // Flag to check if there is a match in any of the td elements
      var hasMatch = false;

      // Loop through the column range that is provided by colStart and colEnd 
      for (j = colStart; j <= colEnd; j++) {
        td = tds[j];
        if (td) {
          // Check if the filter is found in any of the td elements
          txtValue = td.textContent || td.innerText;
          if (txtValue.toUpperCase().indexOf(filter) > -1) {
            hasMatch = true;
            // Exit the loop if a match is found in any td element
            break; 
          }
        }
      }

      // Set the display property based on whether there is a match
      tr[i].style.display = hasMatch ? "" : "none";
    }
  }
</script>
<input id="clientsSearch" type="search" onkeyup="tableSearch('clientsSearch', 'clientsTable', 1, 3)" />
<table class="table" id="clientsTable">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Email</th>
      <th>Phone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>John Doe</td>
      <td>john.doe@example.com</td>
      <td>(123) 456-7890</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Jane Smith</td>
      <td>jane.smith@example.com</td>
      <td>(234) 567-8901</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Mike Johnson</td>
      <td>mike.johnson@example.com</td>
      <td>(345) 678-9012</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Alice Williams</td>
      <td>alice.williams@example.com</td>
      <td>(456) 789-0123</td>
    </tr>
    <tr>
      <td>5</td>
      <td>Bob Brown</td>
      <td>bob.brown@example.com</td>
      <td>(567) 901-2345</td>
    </tr>
  </tbody>
</table>
```
