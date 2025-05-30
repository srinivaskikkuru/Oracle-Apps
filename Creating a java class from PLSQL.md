# User Guide: `ItemTrackDirList` Java Class for PL/SQL Integration

This guide provides a detailed explanation of the `ItemTrackDirList` Java class, designed to be compiled and used within an Oracle PL/SQL environment. It outlines the class's purpose, functionality, and how to integrate it with your PL/SQL code.

## 1. Introduction

The `ItemTrackDirList` Java class is designed to interact with the file system from within an Oracle database. Its primary function is to list the contents of a specified directory and identify files whose names contain the keywords "update" or "insert" (case-insensitive). These identified filenames are then inserted into a database table named `XXITEMTRACK_DIR_LIST`.

## 2. Class Definition and Purpose

The Java class is defined as follows:

```java
CREATE OR REPLACE AND COMPILE JAVA SOURCE NAMED "ItemTrackDirList" [cite: 1]
 AS import java.io.*; [cite: 1]
import java.sql.*; [cite: 1]

public class ItemTrackDirList [cite: 2]
{
  public static void getList(String directory) [cite: 2]
                   throws SQLException [cite: 2]
  {
    File path = new File( directory ); [cite: 2]
    String[] list = path.list(); [cite: 3]
    String element; [cite: 3]

    for(int i = 0; i < list.length; i++) [cite: 3]
    {
      element = list[i]; [cite: 3]
      if ((element.toLowerCase().contains("update")==true)||(element.toLowerCase().contains("insert")==true)) [cite: 4]
      {
        #sql { INSERT INTO XXITEMTRACK_DIR_LIST (FILENAME) [cite: 4]
                VALUES (:element) }; [cite: 4]
      }
    }
  }
}