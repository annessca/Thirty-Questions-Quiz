# Thirty Questions Quiz App

This quiz application is based on the sample quiz app tutorial in [Sitepoint](https://www.sitepoint.com/simple-javascript-quiz/) by Yaphi Berhanu. It will be presented somewhat differently with added functionalities. I hope you have fun learning to build this quiz application alongside the tutorial.

## The HTML 

So we'll start by creating an html file. I will name it `quiz.html`

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <title>Thirty Questions Quiz</title>
        <meta name="description" content="A Quiz application based on the sitepoint example">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="style.css">
    </head>
    <body>
        <h1>Thirty questions from the Bible's book of Esther</h1>
        <div class="quiz-container">
            <p id="server-error"></p>
            <div id="quiz"></div>
        </div>
        <br>
        <!-- <button id="previous">Previous Question</button>
		<button id="next">Next Question</button>
		<button id="submit">Submit Quiz</button> -->
        <div id="results"></div>
        <script src="script.js"></script>
    </body>
</html>
```
As you can see, I have included the tags to insert the Javascript and CSS files where you have `<link rel=""stylesheet href="style.css">` and `<script src="script.js"></script>`. You can also see some lines of tags that have been commented out. We will need them later.

## Declaring variables

The next thing you have to do is create Javascript file `script.js`, and write the following code. They are the variables that we will be using in our code.

```javascript
// script.js

let fetchedAPI;
let correctAnsCount = 0;
let currentSlide = 0;
const quizRender = document.getElementById("quiz");

const resultsDisplay = document.getElementById("results");
const submitButton = document.getElementById("submit");

const previousButton = document.getElementById("previous");
const nextButton = document.getElementById("next");

```

## The XMLHttpRequest

The unique thing about this quiz is that we will be calling our questions, options, and correct answers from a mock API. You can find a link to the json file hosted on my Github pages account [here](https://annessca.github.io/simulated-api-files/questions.json). So lets write an `XMLHttpRequest` function to call read data from our API.

Add the following code at the end of your Javascript file

```javascript
// script.js

const readFromJson = (jsonFile, implement) => {
    let extractFile = new XMLHttpRequest();
    extractFile.overrideMimeType("application/json");
    extractFile.open("GET", jsonFile, true);
    extractFile.onreadystatechange = () => {
        if (extractFile.readyState === 4 && extractFile.status == '200') {
            implement(extractFile.responseText);
        } 
    }
    extractFile.send();  
}

```
The function `readFromJson` uses the `new XMLHttpRequst` method to fetch the json file from the API with the parameters `jsonfile` and `implement` as the URL and the callback.

## Building the user interface

Now let's add a new function `quizLogic` that should contain all of the quiz's logic at the bootom of `script.js`.

```javascript
// script.js

const quizLogic = () => {
    const userInterface = [];
    try { 
        readFromJson("https://annessca.github.io/simulated-api-files/questions.json", (questions) => {
            let data = JSON.parse(questions);
            fetchedAPI = data.quizQuestions;
            fetchedAPI.forEach((questionObject, objectIndex) => {
                const options = [];
                for (option in questionObject.answers) {
                    options.push(
                        `<label>
                            <input type="radio" name="question${objectIndex}" value="${option}">
                            ${option} : ${questionObject.answers[option]}
                        </label>`
                    )
                }
        
                userInterface.push(
                    `<div class="question-object">
                        <div class="object-question">${questionObject.question}</div>
                        <div class="object-answers">${options.join("")}</div>
                    </div>`
                );
            });
            quizRender.innerHTML = userInterface.join("");
        });
    }
    catch(error) {
        document.getElementById("server-error").innerHTML = `An unexpected ${error.name} occured - ${error.message}.`;
    }
}
quizLogic();

```
### Code explanation

The implementation of `readFromJson` function happens within a `try` block where the API data is received and assigned the `fetchedAPI` variable.

The `userInterface` array is used to save all the elements and objects used the build the quiz on the browser window. The `options` array which is later pushed into the `userInterface` array distinctly contains the quiz options.

The joined outcome of the `userInterface` array is assigned to `quizRender.html` as a value.



At this point, the quiz should be fully displayed on the browser while I walk you rhough the code so far. We can make it look better by adding some style. Create a file `style.css` and add the following:

```css
/** style.css */

@import url(https://fonts.googleapis.com/css?family=Work+Sans:300,600);

body{
    font-size: 20px;
    font-family: 'Work Sans', sans-serif;
    color: #333;
    font-weight: 300;
    text-align: center;
    background-color: #f8f6f0;
}
h1{
  font-weight: 300;
  margin: 0px;
  padding: 10px;
  font-size: 20px;
  background-color: #444;
  color: #fff;
}
.object-question{
  font-size: 30px;
  margin-bottom: 10px;
}
.object-answers {
  margin-bottom: 20px;
  text-align: left;
  display: inline-block;
}
.object-answers label{
  display: block;
  margin-bottom: 10px;
}

```

Rendering the quiz on the browser is not exactly the entire idea. We are about to add functionalities that actually make it a working quiz.

## Manipulate questions and options

Right under the line 
```
quizRender.innerHTML = userInterface.join("");
```
Add to the rest of the following variables.

```javascript
// script.js

const optionsList = document.querySelectorAll(".object-answers");
const slides = document.querySelectorAll(".question-object");

```
We will need them for the next function `computeOptions`. Go ahead and add the following code right under the last two variable declarations(...constants actually).

```javascript
// script.js

const computeOptions = () => {
    fetchedAPI.forEach((questionObject, objectIndex) => {
        const distinctOption = optionsList[objectIndex];
        const selector = `input[name=question${objectIndex}]:checked`;
        const chosenAnswer = (distinctOption.querySelector(selector) || {}).value;
        if (chosenAnswer === questionObject.correctAnswer) {
            correctAnsCount = CorrectAnsCount + 1;
            optionsList[objectIndex].style.color = 'purple';
        } else {
            optionsList[objectIndex].style.color = 'red'
        }
    });
    resultsDisplay.innerHTML = `${correctAnsCount} out of ${fetchedAPI.length}`;
};

```

### Code explanation

The `optionsList` constant is an array of all elements with the `.objec-answers` class, while `slides` is the same for the `question-object` class. These constants help us reach each member of the questions as well the answer options.

Using the `forEach()` method, we get to select all options by their index in `distinctOption` and the unique name of each radio-button that is checked in `selector` to identify a user's `chosenAnswer`. The `chosenAnswer` is compared with the correct answer in the question to determine the score count in `correctAnsCount`.

The last function is not yet executed because it needs to be called on the `Submit` button. We'll make that happen soon enough. 

## Implementing question slides

Add this last set of functions right after 
```
resultsDisplay.innerHTML = `${correctAnsCount} out of ${fetchedAPI.length}`;
```


```javascript
// script.js

const showSlide = (n) => {
    slides[currentSlide].classList.remove("active-question");
    slides[n].classList.add("active-question");
    currentSlide = n;
            
    if (currentSlide === 0) {
        previousButton.style.display = "none";
    } else {
        previousButton.style.display = "inline-block";
    }
            
    if (currentSlide === slides.length - 1) {
        nextButton.style.display = "none";
        submitButton.style.display = "inline-block";
    } else {
        nextButton.style.display = "inline-block";
        submitButton.style.display = "none";
    }
}
showSlide(0);

```

### Code explanation

The display of our different buttons-(previous, next, and submit) are controlled in the `showSlide` function. The function is called with 0 as an argument to begin the slides from the first question.

Finally, uncumment all the button tags in your `quiz.html` file and add these last two functions and events to your `script.js`.

```javascript
// script.js

const showNextSlide = () => {
    showSlide(currentSlide + 1);
}
        
const showPreviousSlide = () => {
    showSlide(currentSlide - 1);
}

submitButton.addEventListener("click", computeOptions);
previousButton.addEventListener("click", showPreviousSlide);
nextButton.addEventListener("click", showNextSlide);

```

### Code explanation

The last two functions `showNextSlide` and `showPreviousSlide` are executed in the `previousButton` and `nextButton` buttons respectively, while `computeOptions` is executed at the click of the `submit` button

Add the following styles to your `style.css` and test the quiz application on your browser.

```css

button{
  font-family: 'Work Sans', sans-serif;
    font-size: 22px;
    background-color: #279;
    color: #fff;
    border: 0px;
    border-radius: 3px;
    padding: 20px;
    cursor: pointer;
    margin-right: 50px;
}
button:hover{
    background-color: #38a;
}

.question-object{
  position: absolute;
  left: 0px;
  top: 0px;
  width: 100%;
  z-index: 1;
  opacity: 0;
  transition: opacity 0.5s;
}
.active-question{
  opacity: 1;
  z-index: 2;
}
.quiz-container{
  position: relative;
  height: 200px;
  margin-top: 40px;
  margin-bottom: 40px;
}

```

There you have it...the quiz app is working properly. You may now see the what the full code looks like.

```javascript
// script.js

let fetchedAPI;
let correctAnsCount = 0;
let currentSlide = 0;
const quizRender = document.getElementById("quiz");

const resultsDisplay = document.getElementById("results");
const submitButton = document.getElementById("submit");

const previousButton = document.getElementById("previous");
const nextButton = document.getElementById("next");

const readFromJson = (jsonFile, implement) => {
    let extractFile = new XMLHttpRequest();
    extractFile.overrideMimeType("application/json");
    extractFile.open("GET", jsonFile, true);
    extractFile.onreadystatechange = () => {
        if (extractFile.readyState === 4 && extractFile.status == '200') {
            implement(extractFile.responseText);
        } 
    }
    extractFile.send();  
}

const quizLogic = () => {
    const userInterface = [];
    try { 
        readFromJson("https://annessca.github.io/simulated-api-files/questions.json", (questions) => {
            let data = JSON.parse(questions);
            fetchedAPI = data.quizQuestions;
            fetchedAPI.forEach((questionObject, objectIndex) => {
                const options = [];
                for (option in questionObject.answers) {
                    options.push(
                        `<label>
                            <input type="radio" name="question${objectIndex}" value="${option}">
                            ${option} : ${questionObject.answers[option]}
                        </label>`
                    )
                }
        
                userInterface.push(
                    `<div class="question-object">
                        <div class="object-question">${questionObject.question}</div>
                        <div class="object-answers">${options.join("")}</div>
                    </div>`
                );
            });
            quizRender.innerHTML = userInterface.join("");

            const optionsList = document.querySelectorAll(".object-answers");
            const slides = document.querySelectorAll(".question-object");

            const computeOptions = () => {
                // resultsDisplay.innerHTML = `${correctAnsCount} out of ${fetchedAPI.length}`;
                fetchedAPI.forEach((questionObject, objectIndex) => {
                    const distinctAnswer = optionsList[objectIndex];
                    const selector = `input[name=question${objectIndex}]:checked`;
                    const choiceAnswer = (distinctAnswer.querySelector(selector) || {}).value;
                    if (choiceAnswer === questionObject.correctAnswer) {
                        correctAnsCount = CorrectAnsCount + 1;
                        optionsList[objectIndex].style.color = 'purple';
                    } else {
                        optionsList[objectIndex].style.color = 'red'
                    }
                });
                resultsDisplay.innerHTML = `${correctAnsCount} out of ${fetchedAPI.length}`;
            };

            const showSlide = (n) => {
                slides[currentSlide].classList.remove("active-question");
                slides[n].classList.add("active-question");
                currentSlide = n;
            
                if (currentSlide === 0) {
                    previousButton.style.display = "none";
                } else {
                    previousButton.style.display = "inline-block";
                }
            
                if (currentSlide === slides.length - 1) {
                    nextButton.style.display = "none";
                    submitButton.style.display = "inline-block";
                } else {
                    nextButton.style.display = "inline-block";
                    submitButton.style.display = "none";
                }
            }
            showSlide(0);

            const showNextSlide = () => {
                showSlide(currentSlide + 1);
            }
        
            const showPreviousSlide = () => {
                showSlide(currentSlide - 1);
            }

            submitButton.addEventListener("click", computeOptions);
            previousButton.addEventListener("click", showPreviousSlide);
            nextButton.addEventListener("click", showNextSlide);
        
        });
    }
    catch(error) {
        document.getElementById("server-error").innerHTML = `An unexpected ${error.name} occured - ${error.message}.`;
    }
}
quizLogic();

```
There is never a limit to what can be done with an existing system. You should try and add more advanced functionalities to improve the perfomance of the quiz app. The more code you write, the more confidence you gain for yourself in programming.
