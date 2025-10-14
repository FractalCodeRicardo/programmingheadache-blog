---
title: "DBeaver Error: Public Key Retrieval is not allowed"
date: 2023-10-31
categories: 
  - "dbeaver"
---

DBeaver is a powerful database tool that is frequently used for managing MySQL databases. One of the common errors in DBeaver is shown in the image

![](images/image-1024x376.png)

This error is easy to fix. Right click and Edit you connection

![](images/image-1.png)

Then go to driver properties and set AllowPublicRetrieval to TRUE

![](images/image-2.png)

Now the error is gone

![](images/image-3.png)
