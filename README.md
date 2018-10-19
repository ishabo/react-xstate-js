

reference:
https://github.com/davidkpiano/xstate
https://github.com/nenti/react-xstate
https://github.com/MicheleBertoli/react-automata



why?
  to integrate xstate into react, providing more powerful state management to react through finite state machines.

  retuirement: good knowledge of react and xstate

# react-xstate-js

A React wrapper for xstate [xstate](https://github.com/davidkpiano/xstate).

## Example 1

```bash
yarn add react-xstate-js
```

```js
import { Machine } from 'react-xstate-js';

const toggleMachine = Machine({
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } }
  }
});

// Interpret the machine however you want.
// Here's a simple (side-effectful) example:
let currentState = toggleMachine.initialState;

function send(event) {
  currentState = toggleMachine.transition(currentState, event);
  console.log(currentState.value);
}

send('TOGGLE'); // 'active'
send('TOGGLE'); // 'inactive'
```

📖 [Read the documentation!](http://davidkpiano.github.io/xstate/docs)

- [Visualizer](#visualizer)
- [3rd-Party Usage](#3rd-party-usage)
- [Why? (info about statecharts)](#why)
- [Installation](#installation)
- [Finite State Machines](#finite-state-machines)
- [Hierarchical (Nested) State Machines](#hierarchical-nested-state-machines)
- [Parallel State Machines](#parallel-state-machines)
- [History States](#history-states)
- [Interpreters](#interpreters)

## Visualizer

**[:new: Preview and simulate your statecharts in the xstate visualizer (beta)!](https://bit.ly/xstate-viz)**

<a href="https://bit.ly/xstate-viz" title="xstate visualizer"><img src="https://i.imgur.com/fOMJKDZ.png" alt="xstate visualizer" width="300" /></a>

## 3rd-Party Usage

With [sketch.systems](https://sketch.systems), you can now copy-paste your state machine sketches as `xstate`-compatible JSON!
1. Create your sketch (example: https://sketch.systems/anon/sketch/new)
2. Click **Export to clipboard...**
3. Select `XState JSON`

## Why?
Statecharts are a formalism for modeling stateful, reactive systems. This is useful for declaratively describing the _behavior_ of your application, from the individual components to the overall application logic.

Read [📽 the slides](http://slides.com/davidkhourshid/finite-state-machines) ([🎥 video](https://www.youtube.com/watch?v=VU1NKX6Qkxc)) or check out these resources for learning about the importance of finite state machines and statecharts in user interfaces:

- [Statecharts - A Visual Formalism for Complex Systems](http://www.inf.ed.ac.uk/teaching/courses/seoc/2005_2006/resources/statecharts.pdf) by David Harel
- [The World of Statecharts](https://statecharts.github.io/) by Erik Mogensen
- [Pure UI](https://rauchg.com/2015/pure-ui) by Guillermo Rauch
- [Pure UI Control](https://medium.com/@asolove/pure-ui-control-ac8d1be97a8d) by Adam Solove
- [Spectrum - Statecharts Community](https://spectrum.chat/statecharts)

## Installation
1. `npm install xstate --save`
2. `import { Machine } from 'xstate';`

## Finite State Machines

<img src="https://imgur.com/rqqmkJh.png" alt="Light Machine" width="300" />

```js
import { Machine } from 'xstate';

const lightMachine = Machine({
  key: 'light',
  initial: 'green',
  states: {
    green: {
      on: {
        TIMER: 'yellow',
      }
    },
    yellow: {
      on: {
        TIMER: 'red',
      }
    },
    red: {
      on: {
        TIMER: 'green',
      }
    }
  }
});

const currentState = 'green';

const nextState = lightMachine
  .transition(currentState, 'TIMER')
  .value;

// => 'yellow'
```

## Hierarchical (Nested) State Machines

<img src="https://imgur.com/GDZAeB9.png" alt="Hierarchical Light Machine" width="300" />

```js
import { Machine } from 'xstate';

const pedestrianStates = {
  initial: 'walk',
  states: {
    walk: {
      on: {
        PED_TIMER: 'wait'
      }
    },
    wait: {
      on: {
        PED_TIMER: 'stop'
      }
    },
    stop: {}
  }
};

const lightMachine = Machine({
  key: 'light',
  initial: 'green',
  states: {
    green: {
      on: {
        TIMER: 'yellow'
      }
    },
    yellow: {
      on: {
        TIMER: 'red'
      }
    },
    red: {
      on: {
        TIMER: 'green'
      },
      ...pedestrianStates
    }
  }
});

const currentState = 'yellow';

const nextState = lightMachine
  .transition(currentState, 'TIMER')
  .value;
// => {
//   red: 'walk'
// }

lightMachine
  .transition('red.walk', 'PED_TIMER')
  .value;
// => {
//   red: 'wait'
// }
```

**Object notation for hierarchical states:**

```js
// ...
const waitState = lightMachine
  .transition({ red: 'walk' }, 'PED_TIMER')
  .value;

// => { red: 'wait' }

lightMachine
  .transition(waitState, 'PED_TIMER')
  .value;

// => { red: 'stop' }

lightMachine
  .transition({ red: 'stop' }, 'TIMER')
  .value;

// => 'green'
```

## Parallel State Machines

<img src="https://imgur.com/GKd4HwR.png" width="300" alt="Parallel state machine" />

```js
const wordMachine = Machine({
  parallel: true,
  states: {
    bold: {
      initial: 'off',
      states: {
        on: {
          on: { TOGGLE_BOLD: 'off' }
        },
        off: {
          on: { TOGGLE_BOLD: 'on' }
        }
      }
    },
    underline: {
      initial: 'off',
      states: {
        on: {
          on: { TOGGLE_UNDERLINE: 'off' }
        },
        off: {
          on: { TOGGLE_UNDERLINE: 'on' }
        }
      }
    },
    italics: {
      initial: 'off',
      states: {
        on: {
          on: { TOGGLE_ITALICS: 'off' }
        },
        off: {
          on: { TOGGLE_ITALICS: 'on' }
        }
      }
    },
    list: {
      initial: 'none',
      states: {
        none: {
          on: { BULLETS: 'bullets', NUMBERS: 'numbers' }
        },
        bullets: {
          on: { NONE: 'none', NUMBERS: 'numbers' }
        },
        numbers: {
          on: { BULLETS: 'bullets', NONE: 'none' }
        }
      }
    }
  }
});

const boldState = wordMachine
  .transition('bold.off', 'TOGGLE_BOLD')
  .value;

// {
//   bold: 'on',
//   italics: 'off',
//   underline: 'off',
//   list: 'none'
// }

const nextState = wordMachine
  .transition({
    bold: 'off',
    italics: 'off',
    underline: 'on',
    list: 'bullets'
  }, 'TOGGLE_ITALICS')
  .value;

// {
//   bold: 'off',
//   italics: 'on',
//   underline: 'on',
//   list: 'bullets'
// }
```

## History States

<img src="https://imgur.com/I4QsQsz.png" width="300" alt="Machine with history state" />

```js
const paymentMachine = Machine({
  initial: 'method',
  states: {
    method: {
      initial: 'cash',
      states: {
        cash: { on: { SWITCH_CHECK: 'check' } },
        check: { on: { SWITCH_CASH: 'cash' } },
        hist: { history: true }
      },
      on: { NEXT: 'review' }
    },
    review: {
      on: { PREVIOUS: 'method.hist' }
    }
  }
});

const checkState = paymentMachine
  .transition('method.cash', 'SWITCH_CHECK');

// => State {
//   value: { method: 'check' },
//   history: State { ... }
// }

const reviewState = paymentMachine
  .transition(checkState, 'NEXT');

// => State {
//   value: 'review',
//   history: State { ... }
// }

const previousState = paymentMachine
  .transition(reviewState, 'PREVIOUS')
  .value;

// => { method: 'check' }
```

## Sponsors

Huge thanks to the following companies for sponsoring `xstate`. You can sponsor further `xstate` development [on OpenCollective](https://opencollective.com/xstate).

<a href="https://tipe.io" title="Tipe.io"><img src="https://cdn.tipe.io/tipe/tipe-logo.svg?w=240" style="background:#613DEF" /></a>

## Interpreters
- [`xstateful` by @avaragado](https://www.npmjs.com/package/@avaragado/xstateful)
