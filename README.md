# Ducks++: Redux Reducer Bundles

<img src="duck.jpg" align="right"/>

This is an enhancement to the original [Ducks](https://github.com/erikras/ducks-modular-redux) proposal. Basically, the concept is keep redux functionality as modular as possible. The original Ducks standard requires that for each module the associated `{actionTypes, actions, reducer}` should be kept in one file. The additional rules in Ducks++ is to define a string constant inside the duck to determine where the state is stored in the store; this crucially allows the selectors to also be defined in the same file. Finally the default export should be an object which includes the `{actionTypes, actions, reducer, selectors, mountPoint}`.

### Example
A duck module for handling error state in an app:

```javascript
// errors.js

import {createSelector} from 'reselect';
import {fromJS, List} from 'immutable';

const STORE_MOUNT_POINT = 'globals/errors';
const ADD_GLOBAL_ERROR = 'UTILS:ADD_GLOBAL_ERROR';
const CLEAR_GLOBAL_ERRORS = 'UTILS:CLEAR_GLOBAL_ERROR';

// Actions
const addGlobalError = error => ({
    type: ADD_GLOBAL_ERROR,
    payload: error
});

const clearGlobalErrors = () => ({
    type: CLEAR_GLOBAL_ERRORS
});

// Selectors
const getGlobalErrors = state => state[STORE_MOUNT_POINT].get('errors');
const selectGlobalErrors = createSelector(
    getGlobalErrors,
    errors => errors.toJS()
);

const initialState = fromJS({
    errors: []
});

//reducer
const reducer = (state = initialState, action = {}) => {
    switch (action.type) {
        case ADD_GLOBAL_ERROR:
            return state.set('errors', state.get('errors').push(action.payload));
        case CLEAR_GLOBAL_ERRORS:
            return state.set('errors', List());
        default:
            return state;
    }
};

// interface
const errors = {
    mountPoint: STORE_MOUNT_POINT,
    actionTypes: {ADD_GLOBAL_ERROR, CLEAR_GLOBAL_ERRORS},
    actionCreators: {addGlobalError, clearGlobalErrors},
    selectors: {selectGlobalErrors},
    reducer
};
export default errors;
```


### Usage

```javascript
// store.js
import { combineReducers } from 'redux';
import * as ducks from './ducks/index';

function mapDucksToReducers(ducks) {
    const reducers = Object.keys(ducks).map(key => {
        const duck = ducks[key];
        return {
            [duck.mountPoint]: duck.reducer
        };
    });
    return Object.assign({}, ...reducers);
}

const rootReducer = combineReducers(mapDucksToReducers(ducks));
export default rootReducer;
```

A react error display component:

```javascript
// Warning.js
import React from 'react';
import {connect} from 'react-redux';
import globalErrors from 'state/errors';

Snackbar.propTypes = {
    label: React.PropTypes.string
};

const Warning = ({dismissAction, errors=''}) => (
    <div>
        {errors}
        <button onClick={dismissAction}>Dismiss</button>
    </div> 
);

Warning.propTypes = {
    dismissAction: React.PropTypes.func.isRequired,
    errors: React.PropTypes.string
};

const mapStateToProps = state => ({
    errors: globalErrors.selectors.selectGlobalErrors(state)
});

const mapDispatchToProps = dispatch => ({
    dismissAction: () => dispatch(globalErrors.actionCreators.clearGlobalErrors())
});

const WarningContainer = connect(
    mapStateToProps, mapDispatchToProps
)(Warning);

export {WarningContainer as default, Warning};
```


---

![C'mon! Let's migrate all our reducers!](migrate.jpg)
> Photo credit to [Airwolfhound](https://www.flickr.com/photos/24874528@N04/3453886876/).

---

