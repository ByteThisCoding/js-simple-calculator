# js-simple-calculator
This is an example implementation of a calculator app. Feel free to also check out:
* Our [YouTube Video](https://www.youtube.com/watch?v=aeS2-QYu02k), where we code this in real time with music.

## Demo the Live Example
The live example can be viewed in action on [this page](https://htmlpreview.github.io/?https://github.com/ByteThisStore/js-simple-calculator/blob/main/index.html).

## Example Requirements
We're going to create a simple calculator which behaves like the Windows calculator app. Let's explicitly outline the behavioral requirements for our app:
* Calculator should evaluate in entry order. In other words: *we don't need to wory about PEMDAS*.
    * For example: 6 + 2 * 3 should be evaluated as 6+2=8, then 8*3 = 24.
    * This is how normal "basic" calculators work.
* We **must not** use ``eval`` to calculate the answer.
    * This could potentially pose a security vulnerability, so we will avoid it.
* It should support decimal numbers.
* It should support the following operations:
    * Addition
    * Subtraction
    * Multiplication
    * Division
* User should be able to use the mouse or keyboard to input numbers and operations.

## Implementation Analysis
For this analysis, we'll split up our concerns into two categories: *ui presentation* and *calculator logic*. In terms of ui presentation, our app will need to:
* Receive user input in the form of buttons clicks and keyboard strokes.
* Present an interface with buttons corresponding to each number and operation.
* Display the number the user is currently typing.
* Display the result when the user hits *calculate*.

In order to accomplish this, we will need to use JavaScript to react to events from the HTML document and keyboard, then update the HTML document when the calculator logic is finished processing the input.

In terms of calculator logic, we will need to:
* Perform a mathematical operation based on a string or token such as "+".
* Keep some type of running history in one of the following ways:
    1. Keep a list of numbers and operations so far, then calculate everything when the user hits *calculate*.
    2. Perform operations each time the user enters a number and operation, then carry over the result.
* Reset the state properly when the user hits *Clear*.

For this example, we will take approach #2 above, where we perform partial operations and carry the result as we go.

## Creating the UI
The first step we will take is to create the HTML elements which will receive input. We will define multiple rows of buttons, each with 4 buttons, except the bottom row which will only have two. Each button will correspond to a number, an operation, or the command to calculate or clear the input. Above the buttons, we will have two display containers, one to display the current number being entered, and the other to display the previous number.
```html
<!-- Display previous number -->
<div id="small-display"></div>
<!-- Display current number -->
<div id="main-display"></div>

<!-- Calculator button rows -->
<div id="calc-buttons">
    <div class="button-row">
        <button class="btn-num">7</button>
        <button class="btn-num">8</button>
        <button class="btn-num">9</button>
        <button class="btn-op">÷</button>
    </div>
    <div class="button-row">
        <button class="btn-num">4</button>
        <button class="btn-num">5</button>
        <button class="btn-num">6</button>
        <button class="btn-op">·</button>
    </div>

    <!-- other button rows omitted for brevity --->

    <!-- bottom calc row -->
    <div class="button-row">
        <button class="btn-calc">Calculate!</button>
        <button class="btn-op">Clear</button>
    </div>
</div>
```

## Responding to User Input
The calcular will need to respond to user clicks on the buttons and user keystrokes. To faciliate this, and the calculator logic, we will create a JavaScript class. On initialization, it will query the buttons and add click listeners, then add keystroke listeners. When the callbacks are invoked, it will direct the input to other methods within the class. In the code below, we've defined the class with a constructor and method ``onButtonInvokation``. The constructor creates event listeners for the buttons and the keystrokes, and in the callback, forwards the data to ``onButtonInvokation``. We will define that method, and other logic, in the following sections.
```javascript
class Calculator {

    /**
     * Initialize the calculator:
     * : listen to button elements
     * : ... other tasks tbd ...
     */ 
    constructor() {
        //get buttons container
        const calcEl = document.getElementById("calc-buttons");

        //listen to button clicks
        const buttons = calcEl.querySelectorAll("button");
        buttons.forEach(button => {
            button.addEventListener("click", (event) => {
                this.onButtonInvokation(event.target.innerText);
            });
        });

        //listen to keyboard clicks
        window.addEventListener('keydown', (e) => {
            this.onButtonInvokation(e.key);
        });
    }

    onButtonInvokation(val) {
        /*** .... tbd .... */
    }
}
```

## Coding the Number Input Logic
With the HTML and event listeners in place, the last thing we'll need to do is implement the core calculator functionality. In order to do so, we will need to handle certain scenarios and states. In terms of number inputs, we have a few states to make sure we account for:
* The user may currently be entering a digit for an integer.
* The user may be entering a digit which comes after the decimal.
* The user may be entering the dot (.), indicating that they are switching from integer digits to decimal digits.

We will need to keep track of which "mode" the number input corresponds to and ensure that when the calculation is made, the digits are correctly placed on the correct sides of the decimal. In the code below, we will be using two variables to keep track of this:
1. **currentNumber:** a string which represents the left side of the decimal. The start value is "0".
1. **currentNumberDecimal:** a string or null which represents the right side of the decimal. If it is null, the overall number will be an integer. If it has a non-null value, the system will treat the overall number as a decimal.

When the time comes to retrieve the full value, we will call a method ``currentNumberToUnifiedString`` to properly combine everything. The code below shows the portions of the JavaScript class relating to this functionality:
```javascript
class Calculator {

    constructor() {
        //we'll store numbers as strings to preserve "0"s
        this.previousNumber = null;
        this.currentNumber = "0";
        this.currentNumberDecimal = null;
    }
    
    /**
     * If number is clicked, update current number
     */ 
    onNumericButtonInvokation(val) {
        if (this.currentNumber === null) {
            this.currentNumber = "" + val;
        } else if (this.currentNumberDecimal === null) {
            this.currentNumber += val;
        } else {
            this.currentNumberDecimal += val;
        }
        console.log("Current number:", this.currentNumber, this.currentNumberDecimal);
    }

    /**
     * Get the current number + decimal part into a single total
     */ 
    currentNumberToUnifiedString() {
        let num = this.currentNumber;
        if (this.currentNumberDecimal !== null) {
            num += "." + this.currentNumberDecimal;
        }
        if (num && num.length > 1 && num.charAt(0) === "0") {
            num = num.substring(1);
        }
        return num;
    }

    /** ... other methods omitted for bevity ... **/
}
```

## Coding the Operation Input Logic
When the user is finished defining the number they want to perform the calculation on, they will select one of the mathematical operations, such as addition, to perform. When this happens, our calculator needs to:
1. Handle the previous value:
    * If there is no previous value, store the current number as the previous value.
    * If there is a previous value, perform the previously entered operation on the current and previous numbers and store that result as the previous value.
2. Clear the current number.
3. Record the operation which was just selected, which will actually be performed when the user hits "Calculate", or when another operation is selected.

When the user clicks "Calculate!", the logic will be similar, with the exception that we will not be pushing a new operation, nor will we be pushing the current number to the previous number. Instead, the final result will be stored as the current number.

```javascript
class Calculator {

    constructor() {
        this.currentCalcOp = null;

        //we'll store numbers as strings to preserve "0"s
        this.previousNumber = null;
        this.currentNumber = "0";
        this.currentNumberDecimal = null;
    }

    /**
     * Handle the current operation
     */ 
    onOperationButtonInvokation(op) {
        switch (op) {
            case "clear":
                this.currentNumber = "0";
                this.currentNumberDecimal = null;
                break;
            case "calc":
            case "Enter":
                this.finishCalc();
                break;
            case ".":
                this.currentNumberDecimal = "";
                break;
            case "neg":
                if (this.currentNumber !== "0") {
                    this.currentNumber = this.currentNumber.indexOf("-") === 0
                        ? this.currentNumber.substring(1)
                        : ("-" + this.currentNumber).replace("-0", "-");
                }
                break;
            default:
                this.pushCalc(op);
                break;
        }
    }

    /**
     * Perform partial calculation
     */ 
    pushCalc(calcOp) {
        let doFlip = true;

        const addVal = parseFloat(this.currentNumberToUnifiedString());
        if (this.previousNumber === null) {
            this.previousNumber = addVal;
        } else {
            switch (this.currentCalcOp) {
                case "+":
                    this.previousNumber += addVal;
                    break;
                case "-":
                    this.previousNumber -= addVal;
                    break;
                case "·":
                case "*":
                    this.previousNumber *= addVal;
                    break;
                case "/":
                case "÷":
                    this.previousNumber /= addVal;
                    break;
                //if default reached, unknown input, so skip
                default:
                    doFlip = false;
                    break;
            }
        }

        if (doFlip) {
            this.currentNumber = null;
            this.currentNumberDecimal = null;
            this.currentCalcOp = calcOp;
            console.log("Partial calc: ", this.previousNumber);
        }
    }

    /**
     * Assign variables differently when the "calculate"
     *  button is clicked
     */ 
    finishCalc() {
        this.pushCalc(null);
        //cast back to string
        this.currentNumber = "" + this.previousNumber;
        this.previousNumber = null;
        console.log("Final calc:", this.currentNumber);
    }
}
```