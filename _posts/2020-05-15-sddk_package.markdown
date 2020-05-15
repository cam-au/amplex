---
layout: post
title:  "sddk python package"
date:   2019-05-15 12:00:00
author: Vojtech
categories: Digital-pondering
---

## Introduction

Soon after we started our SDAM project, we began to discuss what platform to choose as a  data storage for our collaborative research projects. To begin with, we considered whether our needs could be fully met by a combination of Google Drive and GitHub. But it revealed that not, at least since Github is not suitable for working with large files and Google Drive is not always easily accessible programmatically and everything there tends to be too much fluid to consider it an appropriate data storage for research purposes. 

Therefore, as researchers based at a Danish research institution, we were really excited by discovering [sciencedata.dk](https://www.deic.dk/en/data_deic_dk), a service for storing, sharing, synchronizing and searching research data managed by [DeiC](https://www.deic.dk/en).  Reading the documentation, we realized that  sciencedata relies on WebDAV API[^1], what means that it might be also  accessed programmatically via any service supporting [HTTP protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_message). Within our project, we work with our data mainly within R and Python,  both of which support the HTTP protocol, so it was a way to go.

Realizing this, I was interested in finding the smoothiest way for exporting any data from my python environment directly to [sciencedata.dk](https://www.deic.dk/en/data_deic_dk) to be instantly accessible for reuse by me or anybody else within our project. With this task at hands, I started to develop the [sddk](https://github.com/sdam-au/sddk_py) python package. To explain my motivation, let me to give you a couple of examples of how I work.

## Workflow

For instance, in one of our empirical research projects, we have developed a series of Jupyter Notebooks,  which we used to extract, transform and clean a dataset of +80,000 Roman inscriptions. I usually run these notebooks on [Google Colab](https://colab.research.google.com/github/tensorflow/examples/blob/master/courses/udacity_intro_to_tensorflow_for_deep_learning/l01c01_introduction_to_colab_and_python.ipynb) online platform and its cloud servers, what means that I can easily access them regardless where I physically sit and what machine I actually use (we also share these notebooks openly on Github in [this repo]( https://github.com/sdam-au/edh_workflow)).

Within Python, I usually work with my data in a form of pandas DataFrame object, which is officially defined as a "two-dimensional size-mutable, potentially heterogeneous tabular data structure with labeled axes (rows and columns). " (see the docs [here](https://pandas.pydata.org/pandas-docs/version/0.23.1/generated/pandas.DataFrame.html)). Using this format, we have a table with more than 80,000 rows and 40 columns which might be easily used for any filtering, grouping or stasticial analysis.

To use this dataset in more than one Jupyter notebook and by other users later on, it has to be exported into a file.  I usually export such data into `.json` files, since they are easily readible by other programming languages, like R. In the example case of Roman inscriptions such a data file has ca 150 MB, but in other projects we do (like [AGT]( https://github.com/sdam-au/AGT_ETL)) it might have even more than 1 GB. 

Since I work on cloud, it does not make too much sense for me to download these files into my machine, have them saved somewhere locally and then to think how to share such files with my collegues. Instead of that, I send them instatly into a team shared folder on sciencedata.dk, where anyone from our team can find it and reuse it following her or his needs. Thus, the data just go from one server (Google servers running Colaboratory) to another (sciencedata.dk), without even touching my personal computer.

Generally speaking, sddk package is nothing more than a series of functions helping us to make this process as much easy as possible, using just one easily memorable piece of code to send the data there (`sddk.write_file()` function) and an analogical code to read them back (`sddk.read_file()` function). It derserves to be describe in more detail.

## Sddk package usage demonstration

To make these functions working, you first need to install & import the package and to specify your sciencedata.dk credentials and the desired target folder for your subsequent requests using  `sddk.configure()` function. This folder might be either your personal, or a shared one owned by some of your colleagues, as in the example below. For a shared folder called `"team_folder"`, owned by user with username `"team_folder_owner@username.inst"`, you would run this:

```python
!pip install sddk # install the package
import sddk # import the packg
conf = sddk.configure("team_folder", "team_folder_owner@username.inst")
```

`sddk.configure()` is designed in a way that any project member can use exactly the same piece of code accessing the shared folder, regardless whether she or he is owner or ordinary user of the folder. Notice  that goes against the internal structure of  sciencedata.dk, which uses a different syntax for folders you are sharing with others ("sharingout" keyword in the HTTP request path) and folders which are shared with you by others ("sharingin" keyword in the HTTP request path).  These things are treated internally in the package, since we consider more  practical to have the exactly same command for both cases (the credentials are entered interatively, so the code for both cases can remain the the same) .

After successful configuration, you can write or read any number of files like in the above.

```python
# define the object:
dataframe_object = pd.DataFrame([("a1", "b1", "c1"), ("a2", "b2", "c2")], columns=["a", "b", "c"]) 
# send the object to file:
sddk.write_file("df_file.json", dataframe_object, conf)
```

Later on, any other user with an access to the same folder can read the file back to python with:

```python
dataframe_object = sddk.read_file("df_file.json", "df", conf)
```

The package enables you export some other types of Python objects into a couple of different formats. For instance, you can use jsons for lists or dictionaries. You can also export dataframes into csv files or feather files. You can also export strings into txt files or images into ong files. The syntax for all these cases is exactly the same as in the command above, with two minor differences:

* in `sddk.write_file()` you have to take care for the filename extension,
* in `sddk.read_file()` you have to always specify the python internal object you want to produce ("df", "str", "list", and "dict" are currently supported python objects).

## Sciencedata.dk and R

As I have already mentioned, next to Python, our project is also using R. While so far do not have a full-fledged R package, we already have a `request()` function for reading files from sciencedata and writing them back there. (see its documentation [here](https://mplex.github.io/cedhar/Sciencedata_dk.html#method-put)) In combination with some other commands, the request function might be used to read a `.json` (exported from pandas dataframe) into R tidiverse **tibble** object and exported back to sciencedata as a json interpretable by Python as pandas dataframe (see the [R example code](https://github.com/sdam-au/R_code/blob/master/EDH_to_tibble.R)). The circle is complete.



[^1]: "WebDAV ([RFC 4918](https://tools.ietf.org/html/rfc4918)) is an extension to [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), the protocol that web-browsers and webservers use to communicate with each other. The WebDAV protocol enables a webserver to behave like a fileserver too, supporting collaborative authoring of web content. WebDAV extends the set of standard HTTP methods and headers to provide the ability to create a file or folder, edit a file in place, copy or move or delete a file, etc. As an extension to HTTP, WebDAV normally uses port 80 for unencrypted access and port 443 (HTTPS) for secure access." [WebDAV: What it is, where it turns up, and its alternatives](https://www.comparitech.com/net-admin/webdav/).

