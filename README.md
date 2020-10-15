# Getting Started

Welcome to your new project.

It contains these folders and files, following our recommended project layout:

File or Folder | Purpose
---------|----------
`bookshopapp/` | content for UI frontends goes here
`approuter`|Handles the incoming UI Requests
`deployer`|deploys the UI app
`db/` | your domain models and data go here
`srv/` | your service models and code go here
`package.json` | project metadata and configuration
`readme.md` | this getting started guide


## Next Steps

- Open a new terminal and run `cds watch` 
- (in VS Code simply choose _**Terminal** > Run Task > cds watch_)
- Start adding content, for example, a [db/schema.cds](db/schema.cds).


## Excercise Day 4 and 5 : Serving Fiori UI using Annotations


1. Create a CAP project
2. Add OData v2 proxy
3. Add UI module using Fiori tools
4. Add Approuter
5. Add Deployer
6. Add Launchpad module
7. Add XSUAA configuration
8. Generate mta.yaml
9. Build & Deploy


## Step 1. Create a CAP project

 1.1 Generate a new CAP project.

     cds init bookshop
   
 1.2 Create db/schema.cds

      using { Currency, managed, sap } from '@sap/cds/common';

      namespace sap.capire.bookshop;

      entity Books : managed {
          key ID       : Integer               @title : 'ID';
              title    : localized String(111) @title : 'Title';
              descr    : localized String(1111)@title : 'Description';
              author   : Association to Authors @title : 'Author';
              stock    : Integer               @title : 'Stock';
              price    : Decimal(9, 2)         @title : 'Price';
              currency : Currency              @title : 'Currency';
      }

      entity Authors : managed {
          key ID           : Integer     @title : 'Author ID';
              name         : String(111) @title : 'Author Name';
              dateOfBirth  : Date        @title : 'Date of Birth';
              dateOfDeath  : Date        @title : 'Date of Death';
              placeOfBirth : String      @title : 'Place of Birth';
              placeOfDeath : String      @title : 'Place of Death';
              books        : Association to many Books on books.author = $self;
      }





1.3. Create srv/cat-service.cds

    using { sap.capire.bookshop as my } from '../db/schema';

    service CatalogService {
        entity Books as projection on my.Books
        entity Autors as projection on my.Authors
    }
    
1.4 Add UI annotations to srv/cat-service.cds


    annotate CatalogService.Books with @(
    UI: {
        HeaderInfo: {
            TypeName: 'Book',
            TypeNamePlural: 'Books',
            Title: { Value: ID },
            Description: { Value: title }
        },
        SelectionFields: [ ID, title, author.name ],
        LineItem: [
            { Value: ID },
            { Value: title },
            { Value: author.name },
            { Value: price },
            { Value: currency_code }               
        ],
        Facets: [
            {
                $Type: 'UI.CollectionFacet',
                Label: 'Book Info',
                Facets: [
                    {$Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Main', Label: 'Main Facet'}
                ]
            }
        ],        
        FieldGroup#Main: {
            Data: [
                { Value: ID },
                { Value: title },
                { Value: author_ID },
                { Value: price },
                { Value: currency_code }               
            ]
        }
    }
    );
    
 1.5. Add db/data/sap.capire.bookshop-Authors.csv as initial data.
 
    ID;name;dateOfBirth;placeOfBirth;dateOfDeath;placeOfDeath
    101;Emily Brontë;1818-07-30;Thornton, Yorkshire;1848-12-19;Haworth, Yorkshire
    107;Charlotte Brontë;1818-04-21;Thornton, Yorkshire;1855-03-31;Haworth, Yorkshire
    150;Edgar Allen Poe;1809-01-19;Boston, Massachusetts;1849-10-07;Baltimore, Maryland
    170;Richard Carpenter;1929-08-14;King’s Lynn, Norfolk;2012-02-26;Hertfordshire, England
    
    
 1.6. Similarly, add db/data/sap.capire.bookshop-Books.csv
 
    ID;title;descr;author_ID;stock;price;currency_code
    201;Wuthering Heights;"Wuthering Heights, Emily Brontë's only novel, was published in 1847 under the pseudonym ""Ellis Bell"". It was written between October 1845 and June 1846. Wuthering Heights and Anne Brontë's Agnes Grey were accepted by publisher Thomas Newby before the success of their sister Charlotte's novel Jane Eyre. After Emily's death, Charlotte edited the manuscript of Wuthering Heights and arranged for the edited version to be published as a posthumous second edition in 1850.";101;12;11.11;GBP
    207;Jane Eyre;"Jane Eyre /ɛər/ (originally published as Jane Eyre: An Autobiography) is a novel by English writer Charlotte Brontë, published under the pen name ""Currer Bell"", on 16 October 1847, by Smith, Elder & Co. of London. The first American edition was published the following year by Harper & Brothers of New York. Primarily a bildungsroman, Jane Eyre follows the experiences of its eponymous heroine, including her growth to adulthood and her love for Mr. Rochester, the brooding master of Thornfield Hall. The novel revolutionised prose fiction in that the focus on Jane's moral and spiritual development is told through an intimate, first-person narrative, where actions and events are coloured by a psychological intensity. The book contains elements of social criticism, with a strong sense of Christian morality at its core and is considered by many to be ahead of its time because of Jane's individualistic character and how the novel approaches the topics of class, sexuality, religion and feminism.";107;11;12.34;GBP
    251;The Raven;"""The Raven"" is a narrative poem by American writer Edgar Allan Poe. First published in January 1845, the poem is often noted for its musicality, stylized language, and supernatural atmosphere. It tells of a talking raven's mysterious visit to a distraught lover, tracing the man's slow fall into madness. The lover, often identified as being a student, is lamenting the loss of his love, Lenore. Sitting on a bust of Pallas, the raven seems to further distress the protagonist with its constant repetition of the word ""Nevermore"". The poem makes use of folk, mythological, religious, and classical references.";150;333;13.13;USD
    252;Eleonora;"""Eleonora"" is a short story by Edgar Allan Poe, first published in 1842 in Philadelphia in the literary annual The Gift. It is often regarded as somewhat autobiographical and has a relatively ""happy"" ending.";150;555;14;USD
    271;Catweazle;Catweazle is a British fantasy television series, starring Geoffrey Bayldon in the title role, and created by Richard Carpenter for London Weekend Television. The first series, produced and directed by Quentin Lawrence, was screened in the UK on ITV in 1970. The second series, directed by David Reid and David Lane, was shown in 1971. Each series had thirteen episodes, most but not all written by Carpenter, who also published two books based on the scripts.;170;22;15;EUR

1.7. Execute cds watch and see the Fiori preview for Books


## Step 2. Add OData v2 proxy

  As we are going to use Fiori Tools, the OData service needs to be adapted to v2 (Fiori tools currently do not fully support OData v4).
  For this, we use @sap/cds-odata-v2-adapter-proxy

 2.1. install @sap/cds-odata-v2-adapter-proxy to your project.

     npm install @sap/cds-odata-v2-adapter-proxy -s
     
 2.2. Create srv/server.js
 
     "use strict";

     const cds = require("@sap/cds");
     const proxy = require("@sap/cds-odata-v2-adapter-proxy");

     cds.on("bootstrap", app => app.use(proxy()));

     module.exports = cds.server;
     
     
## Step 3. Add UI module using Fiori tools

   let’s add UI module using Fiori tools.
   
  3.1. Generate Fiori Project
  
   1. Press Ctrl + Shift + P and launch Application Generator.

   2. Select SAP Fiori elements application.

   3. Select List Report Object Page

   4. Select Connect to an OData Source as Data source, and specify your local CAP OData URL:  http://localhost:4004/v2/catalog/  click next

   5. Select “Books” as Main Entity and click next

   6. Type below information.

   
      Module Name|bookshopapp
      ---------|----------
      Title|Bookshop App
      Namespace|demo
      Description|Bookshop App
      Project Folder|Your CAP project’s root folder
      
      
   Once you press Finish, Fiori elements app will be generated.
   
   
  7. Move to bookshopapp folder you’ve just created and run the app
  
    npm start
    
  3.2. Make adjustments for Cloud Foundry
 
   We have to make some adjustments so that this Fiori app can be deployed as part of MTA.

 

       1. Create bookshopapp/webapp/xs-app.json.

       {
           "welcomeFile": "flpSandbox.html",
           "routes": [{
             "source": "^(.*)",
             "target": "$1",
             "authenticationType": "xsuaa",
             "service": "html5-apps-repo-rt"
           }]
         }


       2. Add ui5-task-zipper as devDependencies to bookshopapp module.

       npm install ui5-task-zipper --save-dev


       3. Configure bookshopapp/package.json.

       Add ui5-task-zipper to ui5>dependencies.

        "ui5": {
         "dependencies": [
          "@sap/ux-ui5-tooling",
          "ui5-task-zipper"
         ]
        }


       4. Replace bookshopapp/ui5.yaml with the following code. What I’ve done are:

       Remove “ui5Theme”
       Add resources section
       Add builder section
       Resulting bookshopapp/ui5.yaml will look like below.

       specVersion: '1.0'
       metadata:
         name: 'bookshopapp'
       type: application
       resources:
         configuration:
           paths:
             webapp: webapp
       server:
         ...
       builder:
         customTasks:
           - name: "ui5-task-zipper"
             afterTask: "uglify"
             configuration:
               archiveName: "uimodule"
               additionalFiles:
                 - webapp/xs-app.json  


3.3. Add Configuration for Launchpad
 

   1. Add crossNavigation settings to bookshopapp/webapp/manifest.json

   This configuration is necessary for executing the app from Fiori Launchpad.

         "sap.app": {
             "id": "demo.bookshopapp",
             "type": "application",
             "i18n": "i18n/i18n.properties",
             "applicationVersion": {
                 "version": "1.0.0"
             },
             "title": "{{appTitle}}",
             "description": "{{appDescription}}",
       "crossNavigation": {
        "inbounds": {
         "intent1": {
          "signature": {
           "parameters": {},
           "additionalParameters": "allowed"
          },
          "semanticObject": "data",
          "action": "display",
          "title": "{{appTitle}}",
          "icon": "sap-icon://search"
         }
        }
       }, 
 

## Step 4. Add Approuter
 
  We are adding Approuter module to route incoming requests and authenticate users.

 
4.1. Create approuter folder at the project root and add package.json inside it.

     {
       "name": "bookshop-approuter",
       "version": "0.0.1",
       "engines": {
         "node": "12.x.x"
       },
       "scripts": {
         "start": "node node_modules/@sap/approuter/approuter.js"
       },
       "dependencies": {
         "@sap/approuter": "^7.1.0"
       }
     }
 

4.2. Add approuter/xs-app.json.

Approuter redirects requests containing path /v2/catalog to srv-api destination, which will be defined later in mta.yaml.
At this point, authenticationType is “none”, because we don’t restrict access to the CAP service.
If you require only users with certain authorization to access the service, you would set authenticationType as “xsuaa”.

     {
         "welcomeFile": "/cp.portal",
         "authenticationMethod": "route",
         "logout": {
           "logoutEndpoint": "/do/logout"
         },
         "routes": [
             {
                 "source": "^/v2/catalog/(.*)$",
                 "authenticationType": "none",
                 "destination": "srv-api",
                 "csrfProtection": false
             }
          ]
       }
 

## Step 5. Add Deployer
 

Deployer module will upload html5 app to HTML5 Application Repository.

 

5.1. Create deployer folder at the project root and add package.json inside it.

     {
         "name": "bookshop-deployer",
         "engines": {
             "node": "12.x.x"
         },
         "dependencies": {
             "@sap/html5-app-deployer": "2.1.0"
         },
         "scripts": {
             "start": "node node_modules/@sap/html5-app-deployer/index.js"
         }
     }


## Step 6. Add Launchpad module
 

This module is necessary for the app to be able to run in CF Launchpad.

 

6.1. Create launchpad folder at the project root and add package.json inside it.

        {
            "name": "launchpad-site-content",
            "description": "Portal site content deployer package",
            "engines": {
              "node": "12.X"
            },
            "dependencies": {
              "@sap/portal-cf-content-deployer": "3.32.0-20200312112659"
            },
            "scripts": {
              "start": "node node_modules/@sap/portal-cf-content-deployer/src/index.js"
            }
          }
 

6.2. Add launchpad/portal-site/CommonDataModel.json. 

Here, you configure Launchpad Tile and Group.

        {
         "_version": "3.0.0",
         "identification": {
           "id": "c9aae627-9601-4a11-83c3-41b94a3c8026-1576776549699",
           "entityType": "bundle"
         },
         "payload": {
           "catalogs": [
          {
            "_version": "3.0.0",
            "identification": {
           "id": "defaultCatalogId",
           "title": "{{catalogTitle}}",
           "entityType": "catalog",
           "i18n": "i18n/i18n.properties"
            },
            "payload": {
           "viz": [
             {
            "id": "demo.bookshopapp",
            "vizId": "data-display"
             }
           ]
            }
          }
           ],
           "groups": [{
                "_version": "3.0.0",
                "identification": {
                  "id": "defaultGroupId",
                  "title": "{{groupTitle}}",
                  "entityType": "group",
                  "i18n": "i18n/i18n.properties"
                },
                "payload": {
                  "viz": [
                    {
                      "id": "demo.bookshopapp-1",
             "appId": "demo.bookshopapp",
             "vizId": "data-display"
                    }
                  ]
                }
              }],
          }
 

6.3. Add launchpad/portal-site/i18n/i18n.properties.

      catalogTitle=Default Catalog
      groupTitle=Default Group
 

## Step 7. Add XSUAA configuration
 

We have specified in bookshopapp/webapp/xs-app.json that HTML5 Application Repository requires user authentication as below, so we need XSUAA service instance bound to Approuter.

    "authenticationType": "xsuaa"
 

7.1. Add xs-security.json to the project’s root.

      {
        "xsappname": "bookshop",
        "tenant-mode": "dedicated",
        "scopes": [
          {
            "name": "uaa.user",
            "description": "UAA"
          }
        ],
        "role-templates": [
          {
            "name": "Token_Exchange",
            "description": "UAA",
            "scope-references": [
              "uaa.user"
            ]
          }
        ]
      }


## Step 8. Generate mta.yaml
 

This is the final step before deploy. We are connecting everything together in mta.yaml.

 

8.1. Add below configuration to package.json, which is at the project root.

It indicates that HANA DB will be used in production, while SQLite DB will be used for development.

        "cds": {
            "requires": {
                "db": {
                    "kind": "hana"
                }
            }
        }


8.2. Generate mta.yaml.

     cds add mta
     
Next,  we need to make some changes to mta.yaml.

 

8.3. Modify bookshop-db resource

As I’m using trial account, I need to change the parameter “service” from hana to hanatrial.

    # ------------------------------------------------------------
     - name: bookshop-db
    # ------------------------------------------------------------
       type: com.sap.xs.hdi-container
       parameters:
         service: hanatrial
         service-plan: hdi-shared
       properties:
         hdi-service-name: ${service-name}
 

8.4. Add Approuter module.

     - name: bookshop-app-router
       type: approuter.nodejs
       path: approuter
       parameters:
         disk-quota: 512M
         memory: 512M
       requires:
         - name: bookshop_uaa
         - name: bookshop_html5_repo_runtime
         - name: bookshop_portal
         - name: srv-api
           group: destinations
           properties:
             name: srv-api
             url: "~{srv-url}"
             forwardAuthToken: true  
 

8.5. Add UI and Deployer modules.

     - name: bookshop_ui_deployer
       type: com.sap.application.content
       path: deployer
       requires:
         - name: bookshop_html5_repo_host
           parameters:
             content-target: true
       build-parameters:
         build-result: resources
         requires:
           - name: bookshopapp
             artifacts:
               - dist/uimodule.zip
             target-path: resources/
     - name: bookshopapp
       type: html5
       path: bookshopapp
       build-parameters:
         builder: custom
         commands:
           - npm install
           - npm run build
         supported-platforms: []
 

8.6. Add Launchpad deployer module.

     - name: bookshop_launchpad_deployer
       type: com.sap.portal.content
       path: launchpad
       deployed-after:
         - bookshop_ui_deployer
       requires:
         - name: bookshop_portal
         - name: bookshop_html5_repo_host
         - name: bookshop_uaa   


8.7. Add the following resources.

      - name: bookshop_uaa
        type: org.cloudfoundry.managed-service
        parameters:
          path: ./xs-security.json
          service-plan: application
          service: xsuaa
      - name: bookshop_html5_repo_runtime
        type: org.cloudfoundry.managed-service
        parameters:
          service-plan: app-runtime
          service: html5-apps-repo
      - name: bookshop_html5_repo_host
        type: org.cloudfoundry.managed-service
        parameters:
          service-plan: app-host
          service: html5-apps-repo
          config:
            sizeLimit: 1
      - name: bookshop_portal
        type: org.cloudfoundry.managed-service
        parameters:
          service-plan: standard
          service: portal


In the end, mta.yaml will look like this.

 

8.8. After you’ve generated mta.yaml, add cds configuration for local development.

       "cds": {
         "requires": {
           "db": {
             "kind": "hana"
           }
         },
         "[development]": {
           "requires": {
             "db": {
               "kind": "sql"
             }
           }
         }
       }
 

## Step 9. Build & Deploy
 

Finally, we deploy the MTA project to Cloud Foundry.

 

9.1. Run below command to build the project.

    mbt build


If you get any error, check the below dependencies in package.json file. If its not there,

    npm install


Below is package.json after upgrade.

    "dependencies": {
        "@sap/cds": "^4",
        "@sap/cds-odata-v2-adapter-proxy": "^1.4.43",
        "express": "^4",
        "@sap/hana-client": "^2.4.177"
    }
 

9.2. Run below command to deploy the project to Cloud Foundry.

Make sure you have logged in to the target CF subaccount.

    cf deploy mta_archives/bookshop_1.0.0.mtar
 

If everything goes well, you’ll see bookshop-app-router running in your space.
       
 
