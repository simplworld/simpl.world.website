---
title: Simpl-React Validation
permalink: /docs/services/simpl-react/forms/validation/
layout: docs
description:
---

# Validation

## Field level validation

All components in `simpl/lib/components/forms/` share the same API for validing the field.

Supported props are:

* `errors`: An array of functions or strings that may return an error message (see [errors, warnings and sanitizers](#errors-warnings-and-sanitizers))
* `warnings`: An array of functions or strings that may return an warning message (see [errors, warnings and sanitizers](#errors-warnings-and-sanitizers))
* `sanitizers`: An array of functions or strings to modify the value entered by the user (eg: lowercase an email address)
* `formatters`: An array of functions to modify the value _displayed_ to the user
* `required`: A boolean indicating if the input is required. Defaults to `false`.
* `name`: Required. The name of the field.

## errors, warnings and sanitizers

These props accepts an array of functions or strings.

Some useful validator strings are:

* `'isAlpha'`
* `'isAlphanumeric'`
* `'isCurrency'`,
* `'isEmail'`,
* `'isDate'`
* `'isISO8601'`
* `'isURL'`

Some useful sanitizers are:

* `'escape'`
* `'trim'`
* `'toDate'`

You can find the full list of the supported strings in the docs of the [validator package](https://www.npmjs.com/package/validator)

When using functions, every function will receive the field's original value (ie: `event.target.value`, a string) and its props as arguments, and should return either a string for the error/warning message, or `null`:

```jsx
const isOldEnough = (value, ownProps) => {
    if (parseInt(value, 10) < 14) {
        return "Only 14yo and older can register to the site."
    }
    return null;
};

<IntegerInput errors={[isOldEnough]} />
```

Sanitizers and formatters works similarly, but they must return the modified value, or the original value in case of error.

## Form-level validation

If you need to validate against multiple fields (for example, you may need to compare fields' values), you can use [`redux-form`](http://redux-form.com/). The appropriate reducer is already installed in the default store, so you can start using it right away.

In short, this involves the following steps:

0. Create a component for your form, decorating it with `reduxForm`
0. Create your own validation function, which will throw `SubmissionError` if necessary.
0. On submit, call `props.handleSubmit(myValidationFunction)`

This is an example of how it would look in practice, by applying the smart vs presentational component pattern:

```jsx
import React from 'react';
import {connect} from 'react-redux';
import {Form} from 'react-bootstrap';
import { Field, reduxForm } from 'redux-form';
import EmailInput from 'simpl/lib/components/forms/EmailInput.react';
import PasswordInput from 'simpl/lib/components/forms/PasswordInput.react';
import {reduxFormPropTypes}  from 'simpl/lib/components/forms/props';

import { SubmissionError } from 'redux-form'


const SubmitValidationForm = (props) => {
    const { error, handleSubmit, pristine, reset, submitting } = props;
    return (
        <Form onSubmit={handleSubmit(props.submitForm)}>
            <div>
                <label htmlFor="email">email</label>
                <Field name="email" component={EmailInput} type="text"/>
            </div>
            <div>
                <label htmlFor="password">pass</label>
                <Field name="password" component={PasswordInput} type="text"/>
            </div>
            {error && <strong>{error}</strong>}
            <button type="submit" disabled={submitting}>Submit</button>
            <button type="button" disabled={pristine || submitting} onClick={reset}>Clear Values</button>
        </Form>
    );
}

SubmitValidationForm.propTypes = reduxFormPropTypes;

// Decorate the form component
const ValidateContactForm = reduxForm({
    form: 'myform' // a unique name for this form
})(SubmitValidationForm);

function mapDispatchToProps(dispatch, ownProps) {
    return {
        submitForm(values) {
            // values passed to handlers are already sanitized and converted to
            // the appropriate type for your convenience.
            const errors = {};
            if (!['john@example.com', 'paul@example.com', 'george@example.com', 'ringo@example.com'].includes(values.email)) {
                errors.email = 'User does not exist';
            }
            if (values.password === '') {
                errors.password = 'password required';
            }
            if (errors == {}) {
                window.alert(`You submitted:\n\n${JSON.stringify(values, null, 2)}`)
            } else {
                errors._error = 'Login failed!';
                throw new SubmissionError(errors);
            }
        }
    }
}
const ValidateContactFormContainer = connect(
    null,
    mapDispatchToProps
)(ValidateContactForm);

export default ValidateContactFormContainer;
```

## Server-side validation

Server-side validation involves calling a procedure or publishing to a topic on the modelservice, and having the modelservice raise a `FormError` instance containg the form name and the error messages.

Subscribe to a topic to validate the form values:

```python
from modelservice.games.scopes.exceptions import FormError

class MyWorld(World):
    @subscribe
    def validate_form(self, values, **kwargs):
        if values['email'] == 'django@example.com':
            raise FormError('myform', {'email': 'user already exists.', '_error': 'Registration Failed.'})
```

Create an action to submit the form and publish to the topic:

```js
import { createAction } from 'redux-actions';
import AutobahnReact from 'simpl/lib/autobahn';


export const submitForm = createAction('SUBMIT_MY_FORM', (world, values) =>
    AutobahnReact.publish(`model:model.world.${world.id}.validate_form`, [values])
);
```

Next, discpatch your action when submitting the form:

```jsx
import React from 'react';
import {connect} from 'react-redux';
import {Form} from 'react-bootstrap';
import { Field, reduxForm } from 'redux-form';
import EmailInput from 'simpl/lib/components/forms/EmailInput.react';
import PasswordInput from 'simpl/lib/components/forms/PasswordInput.react';
import {reduxFormPropTypes}  from 'simpl/lib/components/forms/props';

import {submitForm} from '../../actions/FormActions';


const SubmitValidationForm = (props) => {
    const { error, handleSubmit, pristine, reset, submitting } = props;
    return (
        <Form onSubmit={handleSubmit(props.submitForm)}>
            <div>
                <label htmlFor="email">email</label>
                <Field name="email" component={EmailInput} type="text"/>
            </div>
            <div>
                <label htmlFor="password">pass</label>
                <Field name="password" component={PasswordInput} type="text"/>
            </div>
            {error && <strong>{error}</strong>}
            <button type="submit" disabled={submitting}>Submit</button>
            <button type="button" disabled={pristine || submitting} onClick={reset}>Clear Values</button>
        </Form>
    );
}

SubmitValidationForm.propTypes = reduxFormPropTypes;

const ValidateContactForm = reduxForm({
    form: 'myform'
})(SubmitValidationForm);

function mapDispatchToProps(dispatch, ownProps) {
    return {
        submitForm(values) {
            dispatch(testForm(ownProps.world, values));
        }
    }
}
const ValidateContactFormContainer = connect(
    null,
    mapDispatchToProps
)(ValidateContactForm);

export default ValidateContactFormContainer;
```
