
# Documentation about PiCachu project 
# Rules and Patterns of the API testing
- [Dictionary](#dict)
- [Objectives of the Test](#objectives)
- [Project description](#description)
- [Project architecture](#arch)
- [Test Approach](#approach)
  - [Test automation](#auto)
  - [Testing new releases](#new_releases)
- [Process of the testing new feature in Agile development model](#process)
  - [Planning](#planning)
  - [Design](#design)
  - [Develop](#develop)
  - [Testing/Review](#review)
- [Resources](#resources)
- [Test Writing Standard](#standard)
- [Non-functional testing](#nonfunc)
  - [Linting](#linting)
- [CI/CD](#ci)
- [Testing Pyramid](#pyramid)


## Dictionary <a name="dict"></a> 
- **API** - An Application Programming Interface (API) is a way for two or more computer programs to communicate with each other.
- **Authentication** - Authentication is the process of verifying the identity of a user or information.
- **ML** - Machine learning (ML) is a type of artificial intelligence (AI) that allows software applications to become more accurate at predicting outcomes without being explicitly programmed to do so.
- **REST** - REST is an acronym for REpresentational State Transfer and an architectural style for distributed hypermedia systems.
- **CRUD** - CRUD refers to the four basic operations a software application should be able to perform – Create, Read, Update, and Delete.
- **Test Plan Requirements Traceability Matrix** - Requirement Traceability Matrix (RTM) is a document that maps and traces user requirement with test cases. 
- **QA** - Quality Assurance
- **Functional testing** - Functional testing is aimed at checking what functions of the software are implemented and how correctly they are implemented.
- **Non-functional testing** - verification of the correctness of non-functional requirements. HOW the software product works is evaluated. 
Examples: Performance, Compatibility (browsers and devices),Localization testing, e.t.c.

## Objectives of the Test<a name="objectives"></a> 
The main task, for which test strategy is written, is to regulate and document important rules for automating the testing of the whole project. Since all developers are involved in this process, this should be described in the Test Plan (Test Strategy) document. Also, this document describes the test strategy for the "PiCachu" project and includes information on what is to be tested, and how the testing is to be accomplished (test methodology). Specifically, this document describes the tests to be performed, the testing schedule, resources required, entry criteria, exit criteria, dependencies, test tools, metrics and the Test Plan Requirements Traceability Matrix. This is a living test plan and must be changed to reflect Core Team needs and requirements as they arise.
The main purpose of this test is to verify the requirements for the "PiCachu" project.

## Project description<a name="description"></a> 
PiCachu is a service that allows you to create your own gallery and collect them. In every gallery you can upload photos and take the uniqueness of them by 4 ML features: colors, emotions, objects and associations. The uniqueness value depends and is calculated based on other photos within the gallery

## Project architecture<a name="arch"></a> 
Our system consists of the 8 micro-services:

- [Backend](https://github.com/TourmalineCore/picachu-api)\
Main API (REST) service for executing CRUD operations and (managing all the actions of it's services) takes a role of the bridge between each ML model services\
Functions:
    - receive photos from the user
    - write photo to S3 storage
    - send photos for processing
    - records information about the photo in the database
    - request processing results from Result Service
- [Frontend](https://github.com/TourmalineCore/picachu-ui)\
Projects GUI made by design in Figma\
Functions:
    - providing an interface to the user
    - communicating with the API
- [Color Service](https://github.com/TourmalineCore/picachu-color-service)\
Service responsible for determining the dominant colors of the photo\
Functions:
    - accepts a photo processing request from the queue
    - get photos from S3 storage
    - determine the dominant colors n the photo
    - transfer results to the queue with results

- [Emotion Service](https://github.com/TourmalineCore/picachu-emotion-service)\
Service responsible for determining the main emotion of the photo\
Functions:
    - accepts a photo processing request from the queue
    - get photos from S3 storage
    - determination of the photo emotion
    - transfer results to the queue with results

- [Object Service](https://github.com/TourmalineCore/picachu-object-service)\
Service responsible for identifying objects in the photo\
Functions:
    - accepts a request to process a photo from the queue
    - obtains a photo from S3 storage
    - identification of objects on the photo
    - transfers the results to the queue with results

- [Association Service](https://github.com/TourmalineCore/picachu-association-service)\
Service responsible for defining associations to the tags defined by previous models\
Functions:
    - accepts a photo processing request from the queue
    - determine associations by tags, which appeared in photos after processing by other models
    - transfer the results to the queue with the results

- [Result Service](https://github.com/TourmalineCore/picachu-models-results-service)\
Service responsible for writing processing results to the database\
Functions:
    - write the result of processing models to the database
    - provide the endpoint to receive processing results

- [Result Checker Service](https://github.com/TourmalineCore/picachu-models-results-checker)\
Service responsible for requesting processing results from Result Service\
Functions:
    - every few seconds, it asks Result Service for photo id that have all tags except association tags
    - checks with its database to see which photo id models have already been sent for processing by the association model
    - queues previously unsent photo id and the results of processing of other models to be processed by the association model
    - records in its database that the photo id has been transferred

## Technologies and Frameworks<a name="tech"></a> 
- REST API - ```Flask (Python)```
- UI - ```React (TypeScript)```
- DBMS - ```PostgreSQL```
- Photo storage - ```S3 storage```
- Message broker - ```RabbitMQ```
- CI/CD - ```GitHub Actions```


## Test Approach<a name="approach"></a> 
For project verification, the traditional testing model will be used in combination with shift-left. Shift-left testing means that QA begins the testing at the prototyping/designing phase. It will verify the design of the system and be used to find any differences with the requirements and to check the usability for the user.

### Test automation<a name="auto"></a> 
During the development phase, developers (backend and frontend) write code via TDD approach, then submit to QA review. QA must link tests to test-cases in the Test Management System. Linking takes place in a previously prepared environment.
    - Tests must execute in CI/CD before and after pull request. 


### Testing new releases<a name="new_releases"></a> 
- **System Verification Test** will be divided into two phases:
    - Unit testing - all the functionality  (modules and components) of the project is running automatically in [CI/CD](#ci) for both the backend and the frontend to ensure the stability of the system
        

    - E2E testing - E2E tests are running on the frontend as the next phase in CI/CD to simulate user interaction with the system as part of happy path scenarios, where the entire user path is tested through the core functionality of the system


## Process of the testing new feature in Agile development model<a name="process"></a> 

### Planning <a name="planning"></a> 
- Conversation where all members of the team creating plan the strategy of releasing the features of the project and ask about its requirements
 - QA-Engineers are analyzing the requirements

### Design <a name="design"></a> 
- Testing usability of designed feature and identifying inconsistencies with requirements
- QA-Engineers write test-cases on a new feature. Test-cases storage in Test Management System.

### Develop <a name="develop"></a> 
- Developers use TDD (Test Driven Development), so it includes writing tests before writing the actual code.

### Testing/Review <a name="review"></a> 
- After the feature moved from the "Review" tab to the "Testing" - QA-Engineers are going to do code-review of the developers via checking the compatibility of the tests they wrote with the actual test-cases and help to optimize the code of the tests via refactoring. If the review was successful and all is well, the QA puts a "like" emotion on the pull request. Also they are running the features locally on their environment for testing. help to optimize the code of the tests via refactoring.
   
- Before merge of a Pull Request of a feature branch to the main branch - feature must successfully pass all the reviews including review from QA-Engineers to verify the tests in the code. 

## Resources<a name="resources"></a> 
Backend Developers - ```Alexey, Maria```\
Frontend Developers - ```Andrey, Artyom, Ekaterina```\
QA Engineers - ```Elijah, Sergio```\
Data Scientists - ```Maria, Anastasia```\
Project Manager - ```Yulia```\
Team Lead - ```Andrey```\
DevOps - ```Maxim```

## Test Writing Standard<a name="standard"></a> 
Since quality assurance depends on the entire team, it is important to describe the rules of test development for their code so that each developer has an understanding and there are no misunderstandings and issues that slow down the work process. The following will describe the rules for writing tests for all sides of development.\
***Go to the links below for the details***
 - [Backend](https://github.com/TourmalineCore/picachu-api/blob/develop/test-guideline-api.md)
 - [Frontend](https://github.com/TourmalineCore/picachu-ui/blob/develop/test-guideline-ui.md)
 
## Non-functional testing <a name="nonfunc"></a> 
In addition to the aspects of functional testing, it is important to mention that testing can also be non-functional and benefit product quality. 

### Linting <a name="linting"></a> 
One example that applies to non-functional testing is linter, which ensures code cleanliness.
What a linter does?
Linter, based on given rules, recycles the code in a second:

- Places the right tabs and spaces;
- Places semicolons and brackets in the right places;
- in HTML, makes a nice nested structure of tags and put the styles of the tags in a separate block in <style>;
- in other languages, may capitalize function names to make them easier to read;
- removes extra spaces and empty lines.



## CI/CD <a name="ci"></a> 
CI/CD plays a critical role in software testing by providing an automated framework for continuous testing and deployment.

- One way to prevent the leakage of low-quality code is to use an [autolinter](#linting) when creating pull requests. 
We set it up and run it as first thing in CI/CD after any commits - we have Pull Request blocked if the linting didn't go through, so the dirty code doesn't get into the main branch and the developer fixes the problem and writes his code nicely.
- Another way to ensure code quality is to run all the tests automatically after every commit. Unit and E2E tests are in the action at that point.

So we must configure autolinter and automatical executing of all the tests in the "PiCachu" pipelines to provide high code quality.

## Testing Pyramid<a name="pyramid"></a> 

- **Unit Tests**\
The first (from the bottom) and most valuable level for reliable support and development of the project in the future of our Testing Pyramid is the Unit Testing that promotes the TDD methodology of writing tests before writing each function or method of endpoints in our project’s case. 

- **Integration Tests**\
The second level of our Testing Pyramid is the Integration Testing that controls the correct connection between the services of the system and the interaction of the system with DataBase. Integration Testing provides the reliability and confidence that our subsystems and modules interact with each other correctly.

- **E2E (or Entrance) Tests**   
The third level of our Testing Pyramid is the E2E (end-to-end) or, as it is often called, Entrance Testing where system requirements are validated and user flow is checked. There we will test the basic scenario of the user’s interaction with the system and only happy path that includes all the main features of the system.






