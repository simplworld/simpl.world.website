#  Input components included with simpl-react

* `TextInput`: An input field that accept custom validators, sanitizers and formatters. Accepts  a `type` prop, which defaults to `"text"`. Defaults: `{sanitizers: ['trim']}`.
    * `EmailInput`: Like `TextInput`, but with default `type="email"`. Fails validation is the value is not an email. Defaults: `{errors: ['isEmail'], sanitizers: ['normalizeEmail']}`
    * `NumberInput`: Like `TextInput`, but with default `type="number"`. Accepts `min`, `max` and `step`. Defaults: `{sanitizers: ['toFloat'], errors: [min, max]}`.
        * `IntegerInput`: Like `NumberInput`, but only accepts integers. Any non-numeric character will fail validation. Defaults: `{errors: ['isInt', min, max], sanitizers: ['toInt']`
        * `FloatInput`: Like NumberInput, but `step` defaults to `0.1`. Defaults: `{errors: ['isFloat', min, max], sanitizers: ['toFloat']}`
        * `DecimalInput`: Like `NumberInput`, but always renders with `{decimalPlaces}` decimal places, which defaults to `2`. Defaults: `{errors: ['isDecimal', min, max], sanitizers: ['toFloat'], formatters: [decimalPlaces]}`
            * `CurrencyInput`: Like `DecimalInput`, but also renders a [Bootstrap AddOn](https://react-bootstrap.github.io/components.html#forms-input-addons) with `{currencySymbol}`, which defaults to `'$'`. Defaults: `{errors: ['isCurrency', min, max], sanitizers: ['toFloat'], formatters: [decimalPlaces]}`
    * `PasswordInput`: Like `TextInput`, but forces `type="password"`. Defaults: `{}`.

## Creating your custom Input components

If you wish, you can create your own component that implements your custom markup or validation by using the `validateField` decorator.

The `validateField` decorator accepts the following options:

* `errors`: An array of validators (strings or functions) that may return an error message. For available strings, see the npm [validator](https://www.npmjs.com/package/validator) package.
* `warnings`: An array of validators (strings or functions) that may return a warning message. For available strings, see the npm [validator](https://www.npmjs.com/package/validator) package
* `sanitizers`: An array of saniters (strings or functions) to modify the field's returned value. For available strings, see the npm [validator](https://www.npmjs.com/package/validator) package
* `formatters`: An array of saniters (functions) to modify the field's rendered value.

The component will receive the following props:

* `validationState`: the final validation state,  according to the validation results. This will be:
    * `'warning'` if there's warnings but no errors
    * `'red'` if there's at least one warning
    * `'success'` if there's no errors or messages
* `messages`: An array of errors and warning messages (strings)
* `inputProps`: An object of `props` to be passed to your final `<input>` component.

And finally, an example:

```jsx
import React from 'react';

import { validateField } from 'simpl/lib/decorators/forms/validates';

// for convenience, proptypes for input components are already defined and importable
import { inputPropTypes } from 'simpl/lib/components/forms/props';


function Input(props) {
  // Render error messages
  const errors = props.messages.map((msg) => <p key={msg}>{msg}</p>);

  return (
    <div className={props.validationState}>
      <input type="text" {...props.inputProps} />
      {errors}
    </div>
  );
}

Input.propTypes = inputPropTypes;

export const TextInput = validateField({
  sanitizers: ['trim'],
})(Input);

export default TextInput;
```
