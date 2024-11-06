Proof of concept-implementation of this paper https://huggingface.co/papers/2309.12499 in java language

**NEWS**: Happy new Year! I'm keeping an eye on the gpt-pilot project, and figure how to apply/combine both. It is something like proposed the ACE framework. https://github.com/daveshap/ACE_Framework/blob/main/ACE_Framework.md where you have layers of agents. In the ACE framwork, the top layers manage higher order of abstraction. The lower layer, called "code monkey" may be the corresponding with codeplan algorithm.


Getting started
===============

I assume you already master some Java... Compile and create fat jar package with all, the command line ui and all the dependencies:

> mvn package -D skipTests

It may generate 2 jars, one of them is the fat jar with all the dependencies. For help, simply run:

> java -jar opencodeplan-1.0-SNAPSHOT-jar-with-dependencies.jar 

You need the llm engine to be up and running: I currently only use open source models with oobatextgen but should work with any compatible like openai rest api (not tested). I always download quantized models from "Thebloke" in huggingface.co.

start oobatextgen with your model, see last section "tested models":

Once started ooba raises up the api on port 5000. Note: In past versions of ooba the rest api was different so It seems that nowdays, the only api that offers is the openai compatible one.

open a terminal and get into the root of a maven project, and then execute the command to operate on the project, for example:

> java -jar <path>/opencodeplan-1.0-SNAPSHOT-jar-with-dependencies.jar -f /nemofinder/DictionarySpanish.java 14 0 -i "Renombra el método getWords por damePalabras"

the -f is to indicate "the block": the portion of code in witch starts to operate, currently only over body methods... actually it's almost the same because currently I send the full content of the source file. In the paper suggest to prune the unnecesary methods but It is complicate to handle de merge of the code once the llm returns.


the -i is to give the instruction in natural language.

**ATTENTION**: this project is experimental so it is most likely that the result will not be as expected.

As a test repository/project I am using another project of mine https://github.com/jmdevall/nemofinder which I have cloned locally.

If you see the code, to start the iterative process it is needed to indicate a block and an instruction. From what the paper says, an instruction can be of type "Diff" or a "natural language instruction".
The block is difficult to specify since the code has not been parsed yet. I have had to create a class to describe the block (look for it once the code has been parsed).

Project structure:
==================
I have tried to follow a clean architecture, in order to be able to change the parts that are not properly part of the algorithm. These external parts are:

- the llm
- the code repository
- the parser/dependency analyzer

In the domain are the clases that represents the data structures that the algorithm manages.

In the application part are the services... essentially the algorithm.

Currently there are only the command line interface, in the future may plan to integrate with vscode, or expose an api rest or integrate as part of another AI assistant project, I don't know.

I have followed the package structure suggested here:
https://tbuss.de/posts/2023/9-how-to-do-the-package-structure-in-a-ports-and-adapter-architecture/

The objective is to separate the parts that belong to the algorithm from those that do not, so that whoever wants to can reuse as much as possible.

FAQ:
===
What is your goal with this project?

The development is being slow since I only dedicate some part of my free time. Personally, I have been trying other projects that use AI and act as assistants or are capable of generating code. For example:

https://github.com/paul-gauthier/aider, https://github.com/genia-dev/GeniA etc.

The fundamental idea of the "codeplan" paper is to take advantage of the information on the project's dependencies to generate the most specific prompt possible. Make small changes little by little throughout the code base.
The difficulty that attendees seem to currently encounter is:
- 1 manage to gather the relevant information to make a change.
- 2 Print the result of the llm as a change back in the repository.
The paper seems to have solved these two problems: it suggests using the language's own semantic information to obtain the dependencies of the parts involved in the code. This, together with the previous changes, would allow us to obtain the perfect prompt. On the other hand, when making small changes that only affect the code of one class at a time, in principle it is easier to transfer those changes back to the repository, while at the same time they serve to categorize the type of change and thus feed back to the prompt in future modifications.

In any case, the conclusion that seems to prevail is that, the smaller the change and the more controlled it is, the easier it is to take advantage of/interpret the result of the LLM, the easier it is to follow a methodical plan that allows working with large projects. In the paper, an algorithm, codeplan, is suggested, but I suggest going further: some other algorithm could be applied to this same idea... for example... the algorithm that follows the Test Driven Development methodology or whatever is dictated by the plan generated by an agent at a higher level to guide development.
In any case, my only intention is that this can serve as help or inspiration for any other project

Why java and not python?
For many reasons:
1 I am more used to working with java
2 Java is a language widely used in projects in the business world. In practice I think it would be the language that could be used the most.
3 Java is a strongly typed language. As the paper indicates, the algorithm is more appropriate and easier to implement in this type of languages.
4 Python seems more suitable for the particular world of artificial intelligence... it might seem that it is easier to implement in python. However, I don't really use anything extraordinary that requires any specialized bookstore. For me, the llm is simply an external web service to which you put a String and it returns another String. The most complex thing in this case is the Java parser and obtaining the dependencies. Furthermore, since I wanted to reuse something already done, there were few options beyond the javaparser library.

Why do you use javaparser and not use tree-setter as a parser as the paper indicates?
That was my first attempt, but I ran into 2 problems:
1) The java version is linked using a native library. In my case I couldn't get it to work, it gave me core dumps and I couldn't find the reason.
2) In addition to the parser, to find the dependencies it is very useful that the library also implements a "typesolver". This is already given to you by javaparser while for tree-setter I think there is nothing.

Of course, if you want to collaborate, participate, suggest things, ask whatever you want... this is a free project
(Note: this is automatic transalation)


**TODO LIST**

- [x] First test running
- [ ] bug in constructed-by relations in fields (fields are pruned. Search alternative
- [ ] Better documentation and explanation so the community can test it
- [ ] The algorithm is executed on another java project. Maybe I should create a folder within the project so that the test projects could be downloaded using git (currently I only have it in another folder locally and for each test I have to restore the repository by hand.
- [x] Currently the repository object can only handle a single source folder for the project. It should be able to handle multiple source directories, for example in maven projects there is a folder src/main/java and another src/test/java. The changes made by the algorithm affect classes of both.
- [x] When starting a model in ooba with exllama, it seems that ooba exposes another different api in port 5000. It seems that it is the api compatible with openai. It would be advisable in that case to make another implementation of the rest service compatible with openai.
- [ ] Implement the oracle. If it is on a maven project, you should be able to invoke it to pass the tests and pass the results of the tests to the prompt.
- [ ] The paper in the method modifications talks about doing an "escaping object" analysis
- [ ] The code responsible for mixing (which is responsible for mixing the revised code into the pruned code). I had not contemplated that the llm would try to change some lines of some unaffected method. The paper does not specify how to do this step.
(Note: this is automatic transalation)

tested models
=============

phind, instruction template: alpaca
> python server.py --model phind-codellama-34b-v2.Q5_K_M.gguf --threads 12 --n_ctx 16384 --api --verbose

dolphin mistrael mixture of experts, template: chatml
This another one has just arrived and seems to be better, but I have doubts...
> python server.py --model dolphin-2.7-mixtral-8x7b.Q5_K_M.gguf --threads 12 --n_ctx 32768 --api --verbose

wizard coder, instruction template: alpaca
> python server.py --model wizardcoder-33b-v1.1.Q5_K_M.gguf --threads 12 --n_ctx 16384 --api --verbose

new: Each model has its own "instruction template", you must select it by it's name, in your config file. There are currently only "alpacha" and "chatml". In the description of the model thebloke says what the instruction template to use.