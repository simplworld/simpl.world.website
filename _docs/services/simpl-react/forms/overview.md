---
title: Simpl-React Forms
permalink: /docs/services/simpl-react/forms/components/
layout: docs
description:
---

# Forms

`Simpl` includes some convenience components from building forms, such as:

* `EmailInput`
* `IntegerInput`
* `FloatInput`
* `DecimalInput`

These components will react to validation:

* their background color will change to green, red or yellow according to the validation results:
    * yellow if there's warnings but no errors
    * red if there's at least one warning
    * green if there's no errors or messages
* Error and/or warning messages will be displayed right below them.

## Support for redux-form

All components have support for [`redux-form`](http://redux-form.com/), which is already included with `simpl`. Just pass the component to the `Field`, along with your custom [validation props](./validation.md).

Additionally, you can add your own `onBlur` and `onFocus` handlers by usign the
`blur` and `focus` prop, respectively:

```jsx
import React from 'react';
import { Form } from 'react-bootstrap';
import { Field, reduxForm } from 'redux-form';

import IntegerInput from 'simpl/lib/components/forms/IntegerInput.react';

import {reduxFormPropTypes} from 'simpl/lib/components/forms/props';

const isOldEnough = (value, ownProps) => {
    if (parseInt(value, 10) < 14) {
        return "Only 14yo and older can register to the site."
    }
};

const blurred = () => {
    console.log('blurred!');
}

const SubmitValidationForm = (props) => {
    const { error, handleSubmit, pristine, reset, submitting } = props;
    return (
        <Form onSubmit={handleSubmit}>
            <div>
                <label htmlFor="age">Age</label>
                <Field
                    name="age"
                    component={IntegerInput}
                    errors={[isOldEnough]}
                    blur={blurred}
                />
            </div>
            <button type="submit">Submit</button>
        </Form>
    );
}

SubmitValidationForm.propTypes = reduxFormPropTypes;

const ValidateRegistrationForm = reduxForm({
    form: 'registrationform' // a unique name for this form
})(SubmitValidationForm);

export default ValidateRegistrationForm;
```
