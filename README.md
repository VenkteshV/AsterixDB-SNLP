
# Using Standford CoreNLP Library 


This document describe the process for using Standford CoreNLP packages with user-defined functions(UDF) in AsterixDB. We assume you have followed the [installation instructions](http://asterixdb.apache.org/docs/0.9.0/install.html) to set up a running AsterixDB instance.

### How To Use
* Clone this repo onto your local machine.
* Build this project(use `mvn install` or `mvn package`).
* The UDF package will be under `target/` directory.
* Copy provided .jar files under `jar`/ to asterix-server zip file (ie. `asterix-server-0.9.0-binary-assembly.zip`). You will need to unzip this asterix-server zip file and drop all jars into `repo`/ folder then zip it back.
    - zip -rg `asterix-server-0.9.0-binary-assembly.zip` `asterix-server-0.9.0-binary-assembly`
* Install external libraries
* Apply UDFs

###Installing External Libraries

We assume you have followed the ​instructions to set up a running AsterixDB instance. Let us refer to your AsterixDB instance by the name "my_asterix".

**Step 1**: Stop the AsterixDB instance if it is in the ACTIVE state.

        $ managix stop  -n my_asterix

**Step 2**: Install the library using Managix install command. Just to illustrate, we use the help command to look up the syntax

        $ managix help  -cmd install
        Installs a library to an asterix instance.
        Options
        n  Name of Asterix Instance
        d  Name of the dataverse under which the library will be installed
        l  Name of the library
        p  Path to library zip bundle

Above is a sample output and explains the usage and the required parameters. Each library has a name and is installed under a dataverse. Recall that we had created a dataverse by the name - "feeds" prior to creating our datatypes and dataset. We shall name our library - "snlplib", but ofcourse, you may choose another name.

You may download the pre-packaged library here​ and place the downloaded library (a zip bundle) at a convenient location on your disk. To install the library, use the Managix install command. An example is shown below.

    $ managix install -n my_asterix -d feeds -l snlp -p <put the absolute path of the library zip bundle here> 

You should see the following message:

    INFO: Installed library snlp

We shall next start our AsterixDB instance using the start command as shown below.

    $ managix start -n my_asterix


You may now use the AsterixDB library in AQL statements and queries. To look at the installed artifacts, you may execute the following query at the AsterixDB web-console.

    for $x in dataset Metadata.Function 
    return $x

    for $x in dataset Metadata.Library
    return $x        

###Applying UDFs

####Creating Input Data Types
The following query creates a dataverse, that acts as a namespace for all datatypes that we also create there after. We assume that these UDFs will be applied to Twitter data for which we expect a specific schema.

    drop dataverse feeds if exists;
    create dataverse feeds;
    use dataverse feeds;


    create type Tweet as open {
        id: int64,
        text : string
    };

####Creating Output Data Types

* Output type for NER
* Output type for Twitter Sentiment

    use dataverse feeds;

    create type NameEntityType if not exists as closed{
        id: int64,
        text: string,
        entities: [string]
    };
    create type TweetSentimentType if not exists as closed{
        id: int64,
        text: string,
        score: int32,
        sentiment: string
    };


###Function Usage

#####Syntax for Sentiment Analysis

        snlp#getSentiment($item)

- Runs analysis on a given text and gives back a score in range of 0-4
- Argument:
    + item: a data record with an attribute `text`
- Return Value:
    + a record of type SentimentType.
- Expected Result:

        {   
            "id": 1, 
            "text": "Today is Friday", 
            "score": 2, 
            "sentiment": "Neutral" 
        }

#####Syntax for Date Recognition

        snlp#getDate($item)

- Runs analysis on a given text and extracts our Date entities
- Argument:
    + item: a data record with an attribute `text`
- Return Value:
    + a record of type NameEntityType.
- Expected Result:

        {   
            "id": 1, 
            "text": "Yesterday was Thursday",
            "entities": [ "Thursday", "Yesterday" ] 
        }

#####Syntax for Location Recognition

        snlp#getLocation($item)

- Runs analysis on a given text and extracts out Person entities
- Argument:
    + item: a data record with an attribute `text`
- Return Value:
    + a record of type NameEntityType.
- Expected Result:

        { 
            "id": 1, 
            "text": "I fly from NYC to London",
            "entities": [ "NYC", "London" ] 
        }

#####Syntax for Person Recognition

        snlp#getName(item)

- Runs analysis on a given text and extracts our Location entities
- Argument:
    + item: a data record with an attribute `text`
- Return Value:
    + a record of type NameEntityType.
- Expected Result:

        { 
            "id": 1, 
            "text": "Obama flips Bush admin's policies",
            "entities": [ "Obama", "Bush" ] 
        }