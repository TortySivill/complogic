# Prolexa #
This repository contains Prolog code for a simple question-answering assistant.
The top-level module is `prolexa/prolog/prolexa.pl`, which can either be run in
the command line or with speech input and output through the
[alexa developer console](https://developer.amazon.com/alexa/console/ask).

The heavy lifting is done in `prolexa/prolog/prolexa_grammar.pl`, which defines
DCG rules for sentences (that are added to the knowledge base if they don't
already follow), questions (that are answered if possible), and commands (e.g.,
explain why something follows); and `prolexa/prolog/prolexa_engine.pl`, which
implements reasoning by means of meta-interpreters.

Also included are `prolexa/prolog/nl_shell.pl`, which is taken verbatim from
Chapter 7 of *Simply Logical*, and an extended version
`prolexa/prolog/nl_shell2.pl`, which formed the basis for the *prolexa* code.

The code has been tested with [SWI Prolog](https://www.swi-prolog.org) versions
7.6.0 and 8.0.3.

## Command-line interface ##
(The code is executed from the `prolexa/prolog` directory.)

```
% swipl prolexa.pl
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.3)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?- prolexa_cli.
prolexa> "Tell me everything you know".
*** utterance(Tell me everything you know)
*** goal(all_rules(_7210))
*** answer(every human is mortal. peter is human)
every human is mortal. peter is human
prolexa> "Peter is mortal".
*** utterance(Peter is mortal)
*** rule([(mortal(peter):-true)])
*** answer(I already knew that Peter is mortal)
I already knew that Peter is mortal
prolexa> "Explain why Peter is mortal".
*** utterance(Explain why Peter is mortal)
*** goal(explain_question(mortal(peter),_8846,_8834))
*** answer(peter is human; every human is mortal; therefore peter is mortal)
peter is human; every human is mortal; therefore peter is mortal
```

## Amazon Alexa and Prolog integration ##
Follow the steps below if you want to use the Amazon Alexa speech to text and
text to speech facilities.
This requires an HTTP interface that is exposed to the web, for which we use
[Heroku](http://heroku.com).

### Generating intent json for Alexa ###
```
swipl -g "mk_prolexa_intents, halt." prolexa.pl
```
The intents are found in `prolexa_intents.json`. You can copy and paste the
contents of this file while building your skill on the
[alexa developer console](https://developer.amazon.com/alexa/console/ask).


### Localhost workflow (Docker) ###
To build:
```
docker build . -t prolexa
```

To run:
```
docker run -it -p 4000:4000 prolexa
```

To test the server:
```
curl -v POST http://localhost:4000/prolexa -d @testjson --header "Content-Type: application/json"
```

### Heroku workflow ###
#### Initial setup ####
Prerequisites:

- Docker app running in the background.
- Installed Heroku CLI (`brew install heroku/brew/heroku` on MacOS).

---

To see the status of your Heroku webapp use
```
heroku logs
```

in the `prolexa` directory.

---

1. Clone this repository
    ```
    git clone git@github.com:So-Cool/prolexa.git
    cd prolexa
    ```

2. Login to Heroku
    ```
    heroku login
    ```

3. Add Heroku remote
    ```
    heroku git:remote -a prolexa
    ```

#### Development workflow ####
1. Before you start open your local copy of Prolexa and login to Heroku
    ```
    cd prolexa
    heroku container:login
    ```

2. Change local files to your liking
3. Once you're done push them to Heroku
    ```
    heroku container:push web
    heroku container:release web
    ```

4. Test your skill and repeat steps *2.* and *3.* if necessary
5. Once you're done commit all the changes and push them to GitHub
    ```
    git commit -am "My commit message"
    git push origin master
    ```

# Prolexa Plus #
Prolexa Plus requires Python 3.6+ and SWI Prolog version 7.6.0+.
Using a Python virtual environment is advised.
Since the Prolog<->Python bridge is quite fragile, you should consider using:

* *Windows Subsystem for Linux* if you are on Windows,
* *Ubuntu Linux*,
* *MacOS*, or
* the provided *Docker image* (see below)

to minimise potential issues.
For more information on how to set up `pyswip` see:

* <https://github.com/yuce/pyswip/blob/master/INSTALL.md> and
* <https://github.com/yuce/pyswip>.

## Installation ##
1. Install Python dependencies
   ```
   pip install -r requirements.txt
   ```
2. Install language models and data
   ```
   python prolexa/setup_nltk.py
   ```
3. Run *Prolexa Plus*
   ```
   PYTHONPATH=./ python prolexa/prolexa_plus.py
   ```

## Docker ##
Instead of a local install, it is possible to run *Prolexa Plus* with the
designated Docker image.

1. Build the *Prolexa Plus* Docker image
   ```
   docker build -t prolexa-plus -f Dockerfile-prolexa-plus ./
   ```
2. Run *Prolexa Plus* via Docker
   ```
   docker run -it prolexa-plus
   ```
