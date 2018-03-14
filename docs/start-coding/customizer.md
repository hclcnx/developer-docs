---
path: 'customizer'
title: 'Customizer'
docsSection: 'Customizer'
---

## IBM Connections Customizer

IBM Connections Customizer is a middleware proxy service that enables the customization of the IBM Connections user experience (UX). 
In essence Customizer acts as a proxy between IBM Connections and the end-user, which gives it the ability to intercept and modify 
requests and responses, and thus customize anything that flows through it, e.g. the behaviour of APIs, the presentation of the user 
interface, etc. This document focuses on customizations of the user interface.

The IBM Connections Customizer model is simple – the service performs customizations by injecting JavaScript, CSS or 
other web resources into the HTML pages returned by IBM Connections in response to end-user requests, i.e. the requests 
generated within standard components like Communities, Profiles, Files, Homepage and so on, as an end-user navigates the apps. 
The customization details, i.e. typically the code that should be inserted and on identified requests, are defined by 
applications stored inside the IBM Connections Application Registry - aka App Reg.  

App Reg is a centralized design repository used to store and retrieve applications that customize and extend a variety of 
different IBM Connections services – where Customizer is just one such service. In the cloud, App Reg is available to 
organization administrators via the Admin > Manage Organization > Organization Extensions menu path. From here it’s 
possible to create and manage Customizer applications. These apps are simply JSON files containing design metadata 
identifying the components that need to be targeted and the actions that need to be performed. Listing 1 is an illustration 
of a rudimentary Customizer application.

```
Listing 1 – Hello World Customizer App

{
  "services": [
  "Customizer"
  ],

  "name": "Simple Customizer Sample",
  "title": "My First Customizer App",
  "description": "Perform a modification to the Connections Homepage",
  "extensions": [
    {
    "name": "Hello World Extension",
    "type": "com.ibm.customizer.ui",
    "path": "homepage",
    "payload": {
      "include-files": [
        "helloWorld/helloWorld.user.js"
        ],
      "include-repo": {
        "name": "global-samples"
        }
      }
     }
   ]
  }
```

The application JSON in Listing 1 requires little explanation.  The following points can be inferred by a 
quick inspection of the code:

- The app is named “Simple Customizer Sample” and it extends the Customizer service
- It contains one extension named  “Hello World Extension” (apps can have many)
- The extension is a customization of the UI (line #11 -  com.ibm.customizer.ui)
- The customization applies to the Connections homepage (line #12 -  homepage)
- A file named helloWorld.user.js is to be injected into the homepage (line #15)
- helloWorld.user.js is retrieved from a global repository of Customizer samples

A more complete summary of the properties used in Listing 2 is shown below:

| Property        | Description           |
| ------------- |-------------|
| Name | String used to identify the extension |
| Title | Short string description - translatable for international audiences |
| Description | Long string description - translatable for international audiences |
| Services | The service(s) with which the application is associated |
| Type | String used to identify the extension point being implemented – required. Valid values are as follows: 
|  | <ul><li>com.ibm.customizer.ui</li><li>com.ibm.customizer.api</li></ul> |
| Path | String value used to identify the component to be customized: 
|  | <ul><li>activities</li><li>blogs</li><li>communities</li><li>files</li><li>forums</li><li>global (Unlike the other path values, global does not represent a real URL path element but is a keyword meaning match all URLs.)</li><li>homepage</li><li>mycontacts</li><li>news</li><li>profiles</li><li>search</li><li>viewer</li><li>wikis</li></ul> |
| **Payload** | |
|  match: **url** | A regular expression used to provide more fine-grained target resource matching, i.e. beyond the broad match specified in the path property |
| match: **user-name** | String used to identify one or more users as the target for the customization - not unique within a given organization |
| match: **user-id** | String used to identify one or more users as the target for the customization. This property is unique within a given organization |
| match: **user-email** | String used to identify one or more users based on email address value   |
| **include-files** | List of files to be inserted into the response for a matched page request |
| include-repo: **name** | String used to identify the repository where the include-files are stored |

### A Closer Look at Customizer Properties
The properties outlined in Listing 2 can be broken down into two categories: 
- Generic App Reg Properties (Properties defined for all App Reg applications across all services)
- Customizer Service Properties (Properties specific to the Customizer service, i.e. everything in the **Payload** section)

In terms of the generic properties, App Reg requires that any application specify **name**, **title**, **description**, **service** and **type** property values. The Application Registry specification does not require the **path** property to be specified when an application is created, but the Customizer service puts it to good use for every request it processes, as will be seen shortly. Ergo, in reality for Customizer applications, a **path** value is required in order for them to work properly.

Of the generic properties outlined in Listing 2, only **type** and **path** merit any further discussion. A **type** value always equates to an extension point defined by a service. At present Customizer only defines two extension points, com.ibm.customizer.ui and com.ibm.customizer.api. The former is a declaration that a given Customizer extension performs a modification to the IBM Connections UI, and thus will be handled in accordance with a prescribed UI extension pattern – for example any **include-files** specified in the **payload** are always injected into the response document. The latter is reserved for future use – suffice to say that as a middleware proxy Customizer is capable of modifying API behaviours, but that use case is not catered for in the current Customizer release.

```
Listing 3 – Examples of IBM Connections URLs

/* homepage */
//w3-connections.ibm.com/homepage/web/updates/#myStream/imFollowing/all 
//w3-connections.ibm.com/homepage/web/updates/#myStream/statusUpdates/all
//w3-connections.ibm.com/homepage/web/updates/#myStream/discover/all
//w3-connections.ibm.com/homepage/web/updates/#atMentions/atMentions
/* communities */
//w3-connections.ibm.com/communities/service/html/ownedcommunities
//w3-connections.ibm.com/communities/service/html/followedcommunities
//w3-connections.ibm.com/communities/service/html/communityinvites
/* files */
//w3-connections.ibm.com/files/app#/pinnedfiles
//w3-connections.ibm.com/files/app#/person/7f37da40-8f0a-1028-938d-db07163b51b2
/* blogs */
//w3-connections.ibm.com/blogs/roller-ui/allblogs?email=joe_schmoe
//w3-connections.ibm.com/blogs/roller-ui/homepage?lang=en_us
/* wikis */
//w3-connections.ibm.com/wikis/home?lang=en-us#!/mywikis?role=editor

```
	
For Customizer applications, the path property value is used to identify a path element in the IBM Connections request URL, which in most use cases corresponds to a standard IBM Connections component. 

Consider the URLs displayed in Listing 3 - these sample URLs follow a clear pattern where the next element after the IBM Connections cloud domain name identifies the Connections component or application handling the request. The possible values of this element map to the path values enumerated in Listing 2, i.e. homepage, communities, files, etc.

It follows that according as http requests flow through Customizer it can query the Application Registry for any extensions relating to a given request URL and reduce the scope of the result set by specifying the particular in-context path value. Thus a typical REST request from Customizer to App Reg for Files customizations might look like this:

``` 
appregistry/api/v3/services/Customizer/extensions?type=com.ibm.customizer.ui&path=files
```

 … which translates as “get all UI extensions registered for the Customizer service that apply to Files”. This should clarify why Customizer extensions must contain both a type and path value. One caveat to note with regard to the path value is the existence of the special global key word. This is designed to address the use case where an extension needs to apply to all requests and it would be clearly inefficient to have to create an extension for every possible path value. For example, should a customer need to display some corporate footer text at the bottom of every page in IBM Connections then a global extension would facilitate that.
 
In response to the request shown above, App Reg returns whatever number of extensions match these criteria, i.e. a single collection of one or more JSON files just like the one shown previously in Listing 1. It is then up to the Customizer service implementation to parse and apply the design metadata contained in the returned extensions – and that is where the payload data comes into play. 

### Processing Payload Properties

As should now be evident, the generic path property provides a coarse means of querying the Application Registry for extensions pertaining to a given IBM Connections component. The optional match properties inside the Customizer payload provide a further means of fine-tuning the filtering of extensions and essentially deciding whether an extension should be applied to a given URL request or not. All payload properties are meaningless to the App Registry – they are always just passed back to the nominated service container (Customizer in this instance) for processing.

### Fine Grained URL Matching

The match url property takes a regular expression and evaluates it against the current URL. If it matches then the extensions is applied. If no match occurs, the extension is not applied. This is a powerful feature as the following code snippets will demonstrate.
Listing 4 shows a Communities extension that has a fine-grained URL match applied on lines 14 – 16. In this case the extension is only applied if the Communities followedcommunities URL is being processed, and so this extension is ignored for other Communities URLs like those shown in Listing 3, i.e. ownedcommunities, communityinvites, etc.

```
Listing 4 – Customizer App With URL Matching
{
 "services": [
   "Customizer"
 ],
 "name": "Communities Customization",
 "title": "UI Customization for Communities I Follow",
 "description": "Sample to modify Connections Communities",
 "extensions": [
   {
	  "name": "Followed Communities Customizer",
     "type": "com.ibm.customizer.ui",
	  "path": "communities",
      "payload": {
        "match": {
            "url": "followedcommunities"
        },
        "include-files": [
            " flipCard/commListCardsFlipStyle.user.js "
        ] ,
        "include-repo": {
            "name": "global-samples"
        }
      }
    }
  ]
 }

```

Similarly, the following fragment shows how a single global extension can be applied to Homepage and Communities but nothing else:

```
Listing 5 – Global Customizer App With URL Matching
…
 "path": "global",
     "payload": {
       "match": {
           "url": "homepage|communities"
      },
     …

```
	
Note: The design of some IBM Connections components like Homepage are based on the Single Page App paradigm. For example, look at the homepage URLs at the top of Listing 3 – all contain hashtags which means that new http requests are not fired as the user navigates around the page. Thus Customizer is not notified for example when a user moves from imfollowing to atmentions.  By contrast this is not the case in Communities when a user moves from ownedcommunities to followedcommunities.  Thus a developer can target individual Communities URLs using the match url property but cannot use the same technique to match the Homepage hashtag URLs. Instead a homepage extension would need to inject a script that would listen for hash change events and respond accordingly. A sample is included in the homepage samples: newsRiverSectioned.user.js . In particular take a look at the handleHashChangeEvent() function contained within.

It’s easy to envisage many other use cases that would require fine-grained match criteria. For instance, if a customer wants to apply a customization to any Files URL that contains a GUID then this can be achieved by setting the path value to “files” and the match url value to “id=[a-z0-9]{8}-([a-z0-9]{4}-){3}[a-z0-9]{12}” – refer back to Listing 3 for an example of such a Files URL.

**Note**: The various braces contained in the regular expression would need to be escaped (i.e. preceded by a backslash character: \) when entered into JSON content stored in App Reg.

### Fine Grained Matching based on the Active End-User

The match property also accepts various user related conditions based on the current user’s name or id. In both cases single or multi-value parameters may be provided, or in JSON parlance a single string value or an array of strings can be specified. The fragment illustrated in Listing 6 shows how a Communities extension can be specifically targeted at specific users based on their user names: Jane Doe and Joe Schmoe in this example: 

```
Listing 6 –Customizer App Targetting Specific Users By Name
…
	  "path": "communities",
      "payload": {
        "match": {
           "user-name":[
	     "Jane Doe",
	     "Joe Schmoe"
	   ]
        },
…
```

It is important to realise that user names are not unique within an organization so it’s possible to inadvertently target unintended users by employing this technique, i.e. any users of the same name will see the extension. To avoid this it is possible to apply a precise filter by using the user-id match property instead. Note that the term “user id” is sometimes referred to as “subscriber id” in the IBM Connection UI and documentation. 

```
Listing 7 –Customizer App Targetting Specific Users By Id
	  "path": "communities",
      "payload": {
        "match": {
           "user-id":[
	     "20071635",
	     "20071656"
	   ]
        },

```

### The Request Life Cycle for IBM Connections Customizer

To summarize what’s been discussed thus far, Customizer is a proxy and all Connections requests and responses flow through it. Customizer queries the App Registry to ascertain if customizations have been registered for components of Connections based on the paths of the URL requests it processes. Whenever App Registry does return application definitions to Customizer, the metadata contained in the JSON payload is used to finally decide whether or not a customization should be applied. This request processing mechanism can be succinctly summarized in Figure 1 as follows:

Figure 1 – IBM Customizer Request Life Cycle

![Figure 1 – IBM Customizer Request Life Cycle](./CustomizerFigure1.png)
 
You have already read about how Customizer generates App Registry queries and how request matching is performed based on the application payload data. The next thing to figure out is how the file resources listed in the include-files property are managed. 

### Include Files for Code Injections

The include-files payload property lists one or more files to be inserted into the Connections http response thus becoming part of the DOM structure loaded in the end-user’s browser. Listing 1 shows a simple single-item value for this parameter: "helloWorld/helloWorld.user.js", where helloWorld is a folder and helloWorld.user.js is a JavaScript file contained within. This raises a number of interesting questions:

1.	Where do these files reside?

For IBM Connections Cloud, any files declared in the include-files property list are stored in one of two locations:

a)	a private IBM GitHub organization (i.e. github.ibm.com - accessible only to IBM)

b)	a public IBM Connections GitHub organization - https://github.com/ibmcnxdev

The include-repo payload property value identifies the name of the actual repository. For example, in Listing 1 and Listing 4 you see an include-repo object with a name value of "global-samples" being used. This is a reference to a repository on github.ibm.com that contains ready-made samples that any IBM Cloud tenant can use in a Customizer app. "Hello World", "FlipCard" and the other samples featured later in the Standard Samples section are all located in this repository.  IBM Customizer resolves the GitHub organization referred to in the JSON markup –i.e. whether it is in the private or public location. IBM has control over the repositories that care created in both locations  so no duplicate names are allowed.

2.	How do they get there?

Customizer assets like the aforementioned global-samples are directly provisioned to github.ibm.com by the IBM Customizer team. Since this is a private GitHub organization you cannot explore it to discover what repositories are available, but you become aware of them through public samples, documentation and other enablement materials (such as this). It is envisaged that an enhanced App Reg IDE may expose these repositories through the UI in a future release.  

On the other hand you can freely explore the assets available on the public IBM Connections Developers GitHub organization (see Figure 2). By default you are free to leverage any Customizer repository within this organization or to collaborate with the IBM Customizer team to create your own repo in this location. This could be a fork of an existing repository or a brand new repo created for you from scratch, depending on your needs.

If you are familiar with GitHub and have a GitHub account then you are already well on your way. If not, then you can start learning about GitHub here using this quick 10 minute guide. Once you know the rudiments, then creating a GitHub account is straight-forward and free for public and open-source projects. 

In order to inject your own include-files into a Customizer app you need a GitHub repository on the public github.com/ibmcnxdev organization. Typically developers have their own repo that they share with IBM – the step by step procedure is as follows:

a.	Share your repo with IBM – add "ibmcndev" as a collaborator

b.	IBM (ibmcnxdev) then creates a fork of your repository under github.com/ibmcnxdev and grants you read access by default.

c.	You can continue to work on your extension using your original repo for your source code activity, but once you are ready to deliver to IBM Cloud you must issue a pull request to IBM. 

d.	IBM merges your pull request once acceptance criteria are met.

e.	Upon merge, the repo files are automatically pushed to IBM Customizer via a webhook. 

f.	Rinse & repeat starting at Step (c) for extension updates.

Step (c) requires you to issue a Pull Request across forks (in GitHub parlance). The key thing to remember is that your original repo which contains the latest changes is always the “head fork”, while the “base fork” must refer to the repo on github.com/ibmcnxdev.

Step (d) involves an initial lightweight summary review by IBM which looks at various aspects of the proposed customization, primarily from a performance, security and documentation standpoint. However ultimate responsibility for the quality and behaviour of the app remains that of the customer who creates or adopts the customization. The review process by IBM provides no guarantee whatsoever of protection against adverse security or performance impacts.

Figure 2 – IBM Connections Developers Organization on GitHub

![Figure 2 – IBM Connections Developers Organization on GitHub](./CustomizerFigure2.png)

TIP: More information on how to integrate your Customizer include files with IBM Connections Cloud is available in video for on opencode4connections.org:

https://opencode4connections.org/oc4c/customizer.xsp?key=ccc-episode2

### Restricting Access to Include-Files

By default the contents of any repository in either GitHub organization are available for use by Customizer apps by any IBM Cloud tenant. This is a very flexible and convenient model but may not always be the desired solution for every situation. Some tenants may prefer to keep the include-files for Customizer apps private to themselves, or restrict usage to a subset of tenants. Different solutions exist to address these needs:

1.	Access Control Lists for Tenant Organizations

Access Control Lists (ACLs) are used to manage access to a particular object. IBM Connections Customizer provides a very simple implementation of an ACL which can control which tenant organizations are allowed to load include files from your repos. All you need to do is to provide an acl.ids file at the root of your project and populate it with the IBM Connections Cloud ids of the tenant organizations to whom you wish to grant access.  

```
Listing 8 –Sample acl.ids file 
60050207
22716730
10034583
```

This is basically a whitelist for tenant access. Once you create an acl.ids file in your repository then only those tenant organizations listed in the file are allowed to use it - all others are denied access. If no acl.ids file exists then all tenants can potentially leverage the repo in their Customizer apps.

2.	Private GitHub Repositories on github.com/ibmcnxdev

GitHub users on a paid GitHub plan have the option of creating private repositories. Private repositories can still be shared with the IBM Connections Developers organization. The private repository will appear in the list of projects under github.com/ibmcnxdev but only administrators of ibmcnxdev will be able to see the contents – i.e. the repo files have no visibility to regular users or to the general public. Even though read access of the source files is restricted via the repository, you will also need to add an acl.ids file should you also wish to prevent runtime access from other tenant organizations.

3.	Private Repositories on github.ibm.com

If you have privacy needs that are not satisfied by the previous two options you can request a private repository for your organization’s include-files on github.ibm.com.  In this situation the JSON definition would typically not contain any include-repo reference as Customizer will resolve the include-files location based on the tenant’s organization id. 
 
### A Peek Inside Some Include-Files

This journey started as most app dev stories do with a reference to a “Hello World” application, the point of which is to jump start the enablement process which the simplest of extensions. So what exactly does the helloWorld.user.js include file do? Listing 8 shows the code – certain variable names and comments have been trimmed for readability in this document but nothing that affects the execution of the script.

```
Listing 9 –Hello World Include File

1 if(typeof(dojo) != "undefined") {
2  require(["dojo/domReady!"], function(){
3   try {
4     // utility function to wait for a specific element to load...
5     var waitFor = function(callback, eXpath, eXpathRt, maxIV, waitTime){
6         if(!eXpathRt) var eXpathRt = dojo.body();
7         if(!maxIV) var maxIV = 10000; // intervals before expiring
8         if(!waitTime) var waitTime = 1;  // 1000=1 second
9         if(!eXpath) return;
10        var waitInter = 0;  // current interval
11        var intId = setInterval( function(){
12          if(++waitInter<maxIV && !dojo.query(eXpath,eXpathRt).length)
13            return;
14          clearInterval(intId);
15          if( waitInter >= maxIV) { 
16            console.log("**** WAITFOR ["+eXpath+"] WATCH EXPIRED!!! interval "+waitInter+" (max:"+ maxIV +")");
17          } else {
18            console.log("**** WAITFOR ["+eXpath+"] WATCH TRIPPED AT interval "+waitInter+" (max:"+maxInter+")");
19            callback();
20          }
21        }, waitTime); // end setInterval()
22     }; // end waitFor()
23
24     // here we use waitFor to wait for the
25     // .lotusStreamTopLoading div.loaderMain.lotusHidden element
26     // before we proceed to customize the page...
27     waitFor( function(){
28        // wait until the "loading..." node has been hidden
29        // indicating that we have loaded content.
30      dojo.query("span.shareSome-title")[0].textContent="Hello World! ";
31     }, ".lotusStreamTopLoading div.loaderMain.lotusHidden");
32
33    } catch(e) {
34        alert("Exception occurred in helloWorld: " + e);
35    }
39  });
36 }
```

For a simple Hello World example, this may appear to be more complicated than expected, but a closer inspection will simplify matters. 

Before perusing the code be aware of the following points:

•	Most of the code in Listing 8 is a re-usable template that any injection code can sit inside

•	Just 1 line of code are needed for the actual Hello World UI update: Line 30 - in bold

•	IBM Connections classic UI uses Dojo so code is injected into a Dojo structured page

The JavaScript code initially validates that Dojo itself is loaded and then uses a standard Dojo utility (domReady) to wait for the DOM to fully load before calling a bound function to perform the customization. Lines 2 – 23 define a function which will wait up to a maximum of 10 seconds for the page to fully load and if successfully loaded within that time period will execute a callback function. If the page does not load within 10 seconds then an error is logged to the JS console.

Figure 3 Hello World Extension for IBM Connections Homepage

![Figure 3 Hello World Extension for IBM Connections Homepage](./CustomizerFigure3.png)

This waitFor()function is thus called passing in the callback function to manipulate the DOM and modify the UI. The interesting part of the callback function (Line 31 as already highlighted) locates a DOM element and assigns “Hello World” as the text content. When this extension is loaded and run by Customizer then the IBM Connections Homepage is modified in the manner shown in Figure 3.

The code injection can be seen by viewing the source of the IBM Connections Homepage in the browser and scrolling to the bottom of the file. The following tag fragment should be evident:

```
Listing 9 –Customizer Script Injection
…
<script type='text/javascript' src='/files/customizer/helloWorld/helloWorld.user.js'> </script>
…
```

TIP: IBM Connections web pages contain a lot of predefined JS variables which can be leveraged by Customizer extensions. For instance, there is an lconn (Lotus Connections) object with many properties defined that any extension script can exploit. Thus on Line #31, replacing "Hello World: " with "Hello " + lconn.homepage.userName + " " would dynamically include the current user in the Homepage customization. The lconn object and others like it should be explored and leveraged by your extensions.

### Standard Samples

Besides Hello World, there are a number of other ready-made Customizer examples to be available for experimentation. The latest samples can always be found in the samples folder of the Customizer GitHub repository:    https://github.com/ibmcnxdev/customizer

Each sample has its own subfolder which contains the App Reg design definition (JSON file) and the resources to be injected to perform the customization (JavaScript, CSS). Take a look at the following examples:

flipcards

This extension provides an alternative rendering for the Communities pages so that a user’s communities can be displayed as flip cards rather than a table of rows. Figure 4 shows a list of three communities with the traditional row based rendering on the left hand side juxtaposed with the flip card layout on the right. Each flip card displays the Communities logo until the user hovers over it whereupon the card is flipped to display the details of the community in question.

Figure 4 Communities Page before and after Flipcard Customization
 
![Figure 4 Communities Page before and after Flipcard Customization](./CustomizerFigure4.png)
 
The flipCard.json file follows the standard App Reg pattern explained already with the Hello World example.  The JavaScript file commListCardsFlipStyle.user.js uses the sample Dojo wrapper to envelope the customization but the code itself is significantly more advanced and serves to give a more real-world indication of the art-of-the-possible with Customizer extensions.

Look for the Toggle Extension control on the Communities page when this customization is applied. Clicking the button allows the user to switch back and forth between the standard row layout and the flip card format.

newsRiver

This extension targets the IBM Connections Homepage and reformats the layout of the activity stream updates by accentuating the space surrounding each entry. Figure 5 shows the Homepage when the newsRiver customization is run – note how the entries display as sections against a pink backdrop. Notice that the Hello World extension is also applied to the Homepage? This shows how multiple App Reg extensions can target the same IBM Connections path - viewing the source of the page will show two JavaScript file injections in this case. 

Figure 5 Multiple Extensions for IBM Connections Homepage

![Figure 5 Multiple Extensions for IBM Connections Homepage](./CustomizerFigure5.png)
 
profiles

The Profiles extension delivers a more sophisticated rendering to the page that is displayed when the user selects the “My Profile” dropdown menu option in IBM Connections. The new UI look and feel is achieved via stylesheet updates. There are two files in the profiles subfolder - the JS file profilesCustomization.js simply inserts a link to the profilesCustomization.css file which does all the work. The new look Profiles page is shown in Figure 6.

Note the inclusion of a new page header graphic, the relocation of action buttons and so forth.

Figure 6 Profile Page Extension
 
![Figure 6 Profile Page Extension](./CustomizerFigure6.png)
 
### Bringing Some Order To Proceedings

All of the samples viewed so far are simple standalone projects. Typically with Customizer applications there is one main entry point, e.g. main.js, and this resource is referenced in the include-files payload property and rendered in the modified HTML output. However the include-files payload property is an array and can contain more than one file reference. The snippet shown in Listing 10 is an example from the enhanced-activity-stream project available on the OpenCode4Connections GitHub repository:

```
Listing 10 –Multiple Include Files
…
"payload": {
        "include-files": [
          "enhanced-activity-stream/core.js",
          "enhanced-activity-stream/scroller.js",
          "enhanced-activity-stream/notifier.js"
        ]
      }
…
```
The three JavaScript files referenced in Listing 10 will be injected in the order they are listed. 

Another factor to bear in mind is that Customizer applications can contain many extensions, even though the samples described here all have just one single extension each. An extension should ideally represent a project which carries out a specific task or a tightly related set of tasks. The include-files referenced in the extension must be contained in a single include-repo, i.e. it is a strict one-to-one mapping. This makes sense from an organizational standpoint. Extensions do specific jobs and the tools for these jobs are typically found in a single dedicated repository. If your application consists of many related tasks and the tools to carry out the work are many and varied then it would make sense for your application to have multiple extensions, where each extension manages a discrete function and maps to a repo designed for that purpose. Be aware though that the extensions are loaded by Customizer in the alphabetical order of the extension names, and not the order in which they are entered into the JSON definition of the application. If your application has multiple extensions and is sensitive to the load order of the include files then you can control this by applying an ordered naming convention to your extensions.

NOTE: The alphabetical order of extensions applies across all applications. For example, you may have two separate apps that target the IBM Connections homepage. The extensions defined within both applications will be sorted as a single alphabetical list by the App Registry and returned to Customizer and then injected in that order into the IBM Connections homepage.

In most cases the ordering of injections does not present a problem. You can view the order at any time by viewing the Connections page source and looking at the <script> tags inserted at the bottom of the page (as shown in Listing 9).  This section simply describes how the injection mechanism works so that you can plan and organize Customizer projects with that information in mind.
 
### Getting Up and Running

The sample customizations discussed in this document are available to any IBM Connections Cloud tenant organization. Applying a sample customization is an easy way to get started with IBM Connections Customizer and help you get familiar with the process. Any sample can be used put through its paces by importing the relevant JSON file into an organization’s Application Registry. So for example, you could take a copy of the helloWorld.json file from the helloWorld samples project published on the Customizer GitHub repo and import it into App Reg as follows:

1.	Go to https://github.com/ibmcnxdev/customizer

2.	Navigate to the  helloWorld.json file and copy/paste the contents to a local file

3.	As Admin user in your IBM Connections Cloud organization go to:

Admin > Manage Organization > Organization Extensions

4.	Click the new Apps Manager link to take you to the Pink Application Registry †

5.	Click New App and then click the Import button once the App Editor appears

6.	Select your local copy of the helloWorld.json

7.	Insert a match criterion like the one in Listing 6 so that the extension is only applied to you:

i.e. match to your user-name, e-mail address or user id

8.	Click Save to save the application into the Application Registry

9.	Refresh the IBM Connections Homepage and verify that the Hello World extension appears

† Note that Step 4 is temporary while the new IBM Connections Pink UI components are being integrated into the cloud.

TIP:  The steps outlined above are covered in an enablement video available online here:

https://opencode4connections.org/oc4c/customizer.xsp?key=ccc-episode1

You can experiment with the other samples in a similar way.  

In reviewing the include files you may have noticed that some samples use a JavaScript filename notation that follows the GreaseMonkey naming convention: somename.user.js.  This is because these customizations were originally developed as GreaseMonkey scripts using browser-based extensions. They were then deployed on the server-side in IBM Connections as Customizer extensions. This option is not only still valid, it is considered a standard practice for developing Customizer apps – i.e. create some new browser extensions using a user script technology like GreaseMonkey for Firefox or TamperMonkey for Chrome. Once you are happy with the local customization then you can submit the resources to IBM for review – i.e. the JavaScript and CSS files you create using GreaseMonkey or TamperMonkey become your Customizer include-files. You can then invoke the customization by creating a Customizer extension (just like the JSON files contained in the standard samples) in the Application Registry.
 
Some Points to Note regarding Customizer Applications

•	Support for Customizer applications follows the same policy as any other customization to the IBM Connections UI – i.e. IBM 

Support can address questions about the customization process, but cannot address questions about the particulars of your customization. 

•	Listing 2 provides the list of supported paths for Customizer at this point in time. This list currently encompasses all core 

IBM Connections apps but does not include the Administration URLs (aka BSS), or related IBM ICS components like IBM Docs, Meetings, Chat or Verse.  The list may be expanded to include these and possibly other components in the future.

•	Customizer extensions are currently restricted to the organization in which they have been added. For example, users from one 

organization may have access to communities in other organizations if they have been so invited, but they would not see any customizations added to such “external” communities. 

### Useful Online References

User Script Technologies:https://greasyfork.org/en

https://tampermonkey.net/

http://www.greasespot.net/

https://zach-adams.com/2014/05/best-userscripts-tampermonkey-greasemonkey/

https://www.lifewire.com/top-greasemonkey-tampermonkey-user-scripts-4134335

IBM Connections Customizer:

https://opencode4connections.org/

https://github.com/ibmcnxdev/customizer
