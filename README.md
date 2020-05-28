# Password-Strength-Checker-React

The password strength estimation library, Zxcvbn, takes a single string (the password) and returns an object with a number of useful options related to the strength of that string. Dropbox uses zxcvbn on their web, mobile, and desktop apps.

To pass a string to the zxcvbn library, we can simply do the following:

```
zxcvbn('hello');
```

Navigate to the root of your new React project, open a terminal window and run the following command to install zxcvbn.

```
$. npm install --save zxcvbn
```

### Creating the Base Password Strength Meter Component

reate a new JavaScript file in our root directory named PasswordStrengthMeter.js. This will be our new React class component.

PasswordStrengthMeter.js

```
import React, { Component } from 'react';
import './PasswordStrengthMeter.css';

class PasswordStrengthMeter extends Component {
  render() {
    return (
      <div className="password-strength-meter">
        I'm a password strength meter
      </div>
    );
  }
}

export default PasswordStrengthMeter;
```

Inside App.js, import our PasswordStrengthMeter component at the top of the file alongside the other import statements. Finally, insert the <PasswordStrengthMeter /> component tag inside the render method of App.js.

App.js

```
import React, { Component } from 'react';
import PasswordStrengthMeter from './PasswordStrengthMeter';
class App extends Component {
  constructor() {
    super();
  }

  render() {
    return (
      <div className="App">
        <PasswordStrengthMeter />
      </div>
    );
  }
}

export default App;
```

### Passing the Password to the Strength Meter Component

Inside App.js, add an input element and have it so that onChange, it sets the password state property to whatever the value that’s being typed into the input:

App.js:

```
import React, { Component } from 'react';
import PasswordStrengthMeter from './PasswordStrengthMeter';
class App extends Component {
  constructor() {
    super();
    this.state = {
      password: '',
    }
  }

  render() {
    const { password } = this.state;
    return (
      <div className="App">
        <div className="meter">
          <input autoComplete="off" type="password" onChange={e => this.setState({ password: e.target.value })} />
          <PasswordStrengthMeter password={password} />
        </div>
      </div>
    );
  }
}

export default App;
```

### Getting a Result from ZXCVBN

This isn’t a great password strength meter. In fact, it’s the opposite of one right now This is where zxcvbn comes in.

In order to evaluate the password coming from the input element, we need to pass our password string prop to the zxcvbn library. This will return an object with a key named ‘score’. Score is an integer between 0 and 4:

result.score      # Integer from 0-4 (useful for implementing a strength bar)

  0 # too guessable: risky password. (guesses < 10^3)

  1 # very guessable: protection from throttled online attacks. (guesses < 10^6)

  2 # somewhat guessable: protection from unthrottled online attacks. (guesses < 10^8)

  3 # safely unguessable: moderate protection from offline slow-hash scenario. (guesses < 10^10)

  4 # very unguessable: strong protection from offline slow-hash scenario. (guesses >= 10^10)
  
  
Create a new constant called testedResult and assign it to the value of zxcvbn being passed the password string.

```
...
  render() {
    const { password } = this.props;
    const testedResult = zxcvbn(password);
    return (
      <div className="password-strength-meter">
        <label
          className="password-strength-meter-label"
        >
          {password}
        </label>
      </div>
    );
  }
  ...
```

### Adding a Progress Element

We’re almost there, but missing one crucial element: the strength meter itself!

The progress HTML element is the perfect use for this. It takes two attributes: value and max.

Insert a new Progress HTML element above the label and pass testedResult.score into the value attribute, and 4 into the max attribute. We’re passing 4 because that’s the highest value returned from the zxcvbn library, so the progress element will be out of 4.

```
...
  
  render() {
    const { password } = this.props;
    const testedResult = zxcvbn(password);
    return (
      <div className="password-strength-meter">
        <progress
          value={testedResult.score}
          max="4"
        />
        <br />
        <label
          className="password-strength-meter-label"
        >
          {password}
        </label>
      </div>
    );
  }
  
  ...
```

### Adding a Better Label

We’re almost at the finish line. Technically, our Password Strength Meter in React is working, but it could be better. We don’t want to display the actual password that’s being typed. Instead, let’s show a handy label telling the user how strong their password is.

To do this, create a new class method inside the component called createPasswordLabel, that takes a single integer parameter, the score, and returns a string, our interpretation of that score (weak, fair, good, etc)

```
...

  createPasswordLabel = (result) => {
    switch (result.score) {
      case 0:
        return 'Weak';
      case 1:
        return 'Weak';
      case 2:
        return 'Fair';
      case 3:
        return 'Good';
      case 4:
        return 'Strong';
      default:
        return 'Weak';
    }
  }
  
  ...
```

Finally, modify the render method so that we’re calling this new method:


```
render() {
    const { password } = this.props;
    const testedResult = zxcvbn(password);
    return (
      <div className="password-strength-meter">
        <progress
          value={testedResult.score}
          max="4"
        />
        <br />
        <label
          className="password-strength-meter-label"
        >
          {password && (
            <>
              <strong>Password strength:</strong> {this.createPasswordLabel(testedResult)}
            </>
          )}
        </label>
      </div>
    );
  }
```


### Styling our Password Strength Meter

I’m a big fan of focusing on the user experience when it comes to creating React components. Our Password Strength Meter is good, but adding some color would really improve the user experience.

Let’s change the color of the progress meter by applying a CSS class to the progress element depending on the return value of the createPasswordLabel method.

```
...

  <progress
    className={`password-strength-meter-progress strength-${this.createPasswordLabel(testedResult)}`}
    value={testedResult.score}
    max="4"
  />
  
...
```

Save the component, and create a new file in the same directory called PasswordStrengthMeter.css. Add the following CSS to it:

```
.password-strength-meter {
  text-align: left;
}

.password-strength-meter-progress {
  -webkit-appearance: none;
  appearance: none;
  width: 250px;
  height: 8px;
}

.password-strength-meter-progress::-webkit-progress-bar {
  background-color: #eee;
  border-radius: 3px;
}

.password-strength-meter-label {
  font-size: 14px;
}

.password-strength-meter-progress::-webkit-progress-value {
  border-radius: 2px;
  background-size: 35px 20px, 100% 100%, 100% 100%;
}

.strength-Weak::-webkit-progress-value {
  background-color: #F25F5C;
}
.strength-Fair::-webkit-progress-value {
  background-color: #FFE066;
}
.strength-Good::-webkit-progress-value {
  background-color: #247BA0;
}
.strength-Strong::-webkit-progress-value {
  background-color: #70C1B3;
}
```

### Wrapping Up

Well, that’s it. I hope you’ve enjoyed following this tutorial to build a password strength meter in react as much as I’ve enjoyed writing it.
