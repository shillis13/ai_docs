# File Format and Directory Structure Conventions

**Version:** 2.0.0
**Last Updated:** 2025-11-03
**Status:** active
**Maintainer:** PianoMan

---

## Table of Contents


---

## Purpose

## Source Data Format Yaml

### Yaml As Source Of Truth



### **Context Optimization** - Yaml Loads On-Demand, Md Auto-Loads (Token Cost Difference)  File Relationships



### ```  When To Read Which Format



### **Read Yaml When

**

### **Read Markdown When

**

### No Processing Needed  Not All Md Files Are Generated



### **Primary Markdown

**

### **Generated Markdown

**

### **Rule Of Thumb

** If there's a `.yml` file with same base name, the `.md` is generated.

## Directory Structure Patterns

### Numbered Directories



### Directories With Numeric Prefixes Indicate Ordering Or Categorization



### **Numbering Scheme

**

### **00-09

** Temporary/transient (inbox, scratch)

### **10-19

** Active work

### **20-29

** Tools and utilities

### **30-39

** Data and resources

### **40-49

** Processed outputs

### **50-59

** Active/current state

### **80-89

** Configuration and settings

### **90-99

** Templates and archives  common_directory_names: |

### **Standard Patterns Across Projects

**

### **Ai-Specific Conventions

**

### `To Desktop Claude/` - Prompt Queue For Desktop  File Naming Conventions



### **Prefixes Indicate Purpose

**

### **Version Suffixes

**

### `* Yyyymmdd.Md` - Date-Stamped Versions  Symlinks For Cross References



## Context Window Optimization

### File Format Impact On Token Usage



### **Markdown Files

**

### **Yaml Files

**

### **Strategy

**

### Can Have 2-3X Longer Conversations With Yaml Approach  File Size Guidelines



### **Auto-Load As Markdown (<10Kb)

**

### **Keep As Yaml (>10Kb)

**

## Tool Selection For File Operations

### When To Use Desktop Commander Vs Filesystem Connector



### **Use Desktop Commander For

**

### **Use Filesystem Connector For

**

### **Both Work For Basic Operations - Desktop Commander Has More Features.**  Reading Large Files



### Desktop Commander

start_search(

## Quick Reference

### File Format Decision Tree



### ```  Directory Navigation Tips



### ```  Common Locations



## Best Practices

### For Claude Instances



### **When Reading Files

**

### **When Writing Files

**

### **When Organizing

**

### 4. Symlink For Cross-References, Don'T Duplicate  For Users Pianoman



### **Maintaining The System

**

### **Optimization

**

## Examples

### Memory Digest Pattern



### Claude

view cli_coordination_v3.yml           # Load when needed

### User

cat cli_coordination_v3.yml.md | less   # Human review

### ```  Protocol Evolution



### ```  Multi Ai Workspace



## Revision History
