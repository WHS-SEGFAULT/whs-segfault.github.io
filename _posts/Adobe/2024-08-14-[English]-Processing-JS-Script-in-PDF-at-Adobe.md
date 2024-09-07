---
layout: post
title: "[English] Processing JS Script in PDF at Adobe"
subtitle: Exploit Knowledge
excerpt_image: https://github.com/user-attachments/assets/07694f1f-b53d-4752-9f64-815c7c0e9bdc
categories: Adobe
tags: [CVE-2021-39863, Adobe, JavaScript, SpiderMonkey]
---


- PDF can have embedded JavaScript code, and Adobe executes this.
- The JS Engine, SpiderMonkey, executes this embedded JS Script using EScript.api in Adobe.
    - JS Engine, SpiderMonkey used by Adobe is implemented in C++.
- If the URL is defined in the PDF, Adobe performs related to URL and Internet access using IA32.api.

![image](https://github.com/user-attachments/assets/07694f1f-b53d-4752-9f64-815c7c0e9bdc)
[Fig 1] The process of handling JS script embedded in PDF at Adobe Acrobat Reader DC

- We can send the two types of URL to Adobe with PDF.
    1. base URL
        - The base URL means the base of the URL such as https://….
        - We can send the base URL to Adobe by setting the object in the PDF.

        ```
        11 0 obj
        <<
        /Base <FEFF68747470733A2F2F7777772E61612E636F6D2F414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141412F>
        >>
        endobj
        ```
        
    2. relative URL
        - The relative URL means the URL comes out after the base URL.
        - We can send a relative URL to Adobe by passing the relative URL as the parameter of functions related to Internet access with JS script embedded in the PDF.
        - We can do this using the functions, `this.submitForm()`, `app.launchURL()`, etc.
        
        ```js
        try {
          this.submitForm('relative URL');
        } catch (err) { }
        ```
