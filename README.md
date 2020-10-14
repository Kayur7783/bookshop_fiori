# Getting Started

Welcome to your new project.

It contains these folders and files, following our recommended project layout:

File or Folder | Purpose
---------|----------
`app/` | content for UI frontends goes here
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


 Step 1. Create a CAP project

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



          
 
