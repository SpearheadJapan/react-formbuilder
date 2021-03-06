# A React Form Builder

This package provides a simple form building solution for single page as well as multi-page forms. Forms are defined in terms of field names, types, and validation rules, and the builder does everything else.

## Installation

Simply `npm` your way to victory:

```
$ npm install react-formbuilder --save
```

or if you like short codes:

```
$ npm i -S react-formbuilder
```

## Online example

An online example/demo can be found on [https://pomax.github.io/react-formbuilder/example/](https://pomax.github.io/react-formbuilder/example/)

## Form definition data

The basic form data looks like this:

```
{
  fieldname1 : fieldDefinitionObject,
  fieldname2 : fieldDefinitionObject,
  ... : ...,
  ...
}
```

Where a field definition object takes the following form:

```
{
  type: ["image"|text"|"textarea"|"choiceGroup"|"checkbox"|"checkboxGroup"|ReactComponentClass],
  label: String data,
  guideText: String data (optional) for adding a lead-in text for the specific field,

  fieldClassname: String data (optional) representing a custom CSS class for this field's form element,
  labelClassname: String data (optional) representing a custom CSS class for this field's label element,
  placeholder: String data (optional)
  defaultValue: default data for this field (optional) String when "type" is checkbox, text, or textarea. Array of strings when "type" is checkboxGroup, choiceGroup, or multiplicity field. When "type" is image, defaultValue should be the file path which will be used to show preview of the default image. Note defaultValue will be ignored for controlled fields (fields with "controller" set),
  metered: boolean (optional),
  optional: boolean (optional),
  options: array of strings for each option when "type" is choiceGroup, checkboxGroup, or a React component that takes "options" (like a ReactSelect component),
  colCount: number of columns to span, if choiceGroup or checkboxGroup type (optional),
  controller: controller object (optional) - see below,
  validator: Single instance of, or array of, validator object(s) - see below,

  multiplicity: number (optional) marking this field as a "there can be more than one of these", with automatic (+)/(-) controls (currently only works with "text" fields). The number provided specifies the default number of fields to show when the form is bootstrapped,
  removeLabel: string (optional) for the "remove" button next to a multiples field,
  addLabel: string (optional) for the "add" button next to a multiples field

  prompt: string (optional) for the "pick a file from your computer" image button
  reprompt: string (optional) for the "pick a different file from your computer" image button, after initial image selection
  helpText: string (optional) help text to show before the initial image selection

  charLimit: number (optional), used by for text/textarea components
  wordLimit: number (optional), used by for text/textarea components

  charLimitText: function(charCount, charLimit) (optional), used to generate a custom character limit text
  wordLimitText: function(wordCount, wordLimit) (optional), used to generate a custom word limit text
}
```

Each field is built off of the supplied information, with additional functionality based on supplying optional definition properties.

Validator objects look like this:

```
{
  error: String data to show when field does not validate,
  validate: function(value) {
    return nothing if 'value' passes validation, else return whatever makes the
    most sense to your codebase, like a string or a new Error("...") or something
    else that acts as "an error occurred" indicator.
  } (optional)
}
```

If a validator object has no custom validate(value) function, a default validator is used that tests whether (value) contains data or not, passing validation if it represents some kind of defined content.

To chain validators, simply use an array of validator objects and the field will not be considered valid unless each validator (including the implied default if only an error is supplied) passes.

Controller objects look like this:

```
{
  name: String matching the fieldname of the controlling field,
  value: value that the field with [controller.name] needs to match to reveal this field
}
```

Controllers are used for things like showing an input textfield when someone selects an "Other" value in a dropdown list, radio group, or checkbox group.

## Using a Component as a field type

Note that when you use the `type` property to refer to a React component as value, refer to the component class:

```
type: ReactTagCloud,
...
```

When used with the form builder, it is assumed the component has an "onChange" property, in line with React's way of handling change events for HTML form elements.

IF you're writing your own components, you can either set any `onChange` handler to point to `this.props.onChange` to automatically fall through to the Form Builder's update management, or you use your own onChange handler, as long as you make sure to call `this.props.onChange` eventually. For example:

```
class CustomThing extends React.Component {
  onChange(evt) {
    // do a thing here first
    // then pass it on!
    this.props.onChange(evt);

    // Or pass it on with an explicitly know value,
    // if you abstracted that during onChange handling.
    this.props.onChange(evt, this.explicitValue);
  }
}

...

const fields = {
  customthing: {
    type: CustomThing,
    label: "Custom declaration",
    ...
  }
}
```

## Using character/words limits

Fields of the "text" and "textarea" type can be given a numerical `charLimit` and `wordLimit` value, which adds additional markup around the fields to show the number of characters/words written as well as permitted, with CSS classes tacked on when the input exceeds the indicated limits.

The markup when using limits is as follows:

```
<fieldset>
  <label>...</label>
  <span class="over-char-limit over-word-limit">
    <textarea/input type="text">
    <span class="char-limit">.../...</span>
    <span class="word-limit">.../...</span>
  </span>
</fieldset>
```

The `over-char-limit` and `over-word-limit` classes kick in when the input exceeds the specified limits, and the spans for character and word typed vs. limits will only be shown if the relevant field value is specified (i.e. if only a `charLimit` is specified, no word limit markup is generated, and vice versa).

Note that text and textarea fields that do not specify any limits will not contain the special wrapper `<span>` code for indicating current value and limit information.

## Components

### `Form`

The `Form` component defines a form based on a form data object using the format explained above.

Note that the `Form` component is **not** an HTML `<form>` element with an associated action, and as such does not come with a submit button. Instead, the form is responsible for aggregating and validating user data and notifying the owning component of data updates. As such, using a `<Form>` requires an owning component to slap on its own button:

```
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      fields: require('./fields'),
      values: {}
    };
  }

  render() {
    var formProps = {
      fields: this.state.fields,
      onUpdate: (evt,name,field,value) => this.onUpdate(evt,name,field,value)
    };

    return (
      <div>
        <Form ref="form" {...formProps} />
        <button onClick={e => this.submitForm(e)}>Submit</button>
      </div>
    );
  }

  onUpdate(evt, name, field, value) {
    let values = this.state.values;
    values[name] = value;
    this.setState({ values });
  }

  submitForm() {
    this.refs.form.validates( (valid, errors, errorElements) => {
      if (!valid) {
        return console.error("boo, form does not pass validation!");
      }
      console.log("yay, we're good to go!", this.state.values);
    });
  }
}
```

#### validation

validation can be triggered, as per the example, by calling the `.validates()` function on the form component. This function takes a callback that is passed three arguments:

```js
validates( function(valid, errors, errorElements) {
  // - valid <boolean>, indicates whether the form is valid in its entirety or not
  // - errors <string[]>, error messages for each problem found
  // - errorElements <string[]>, the field names for each problematic element
});
```

In this callback, the `errors` and `errorElements` are aligned arrays, such that `error[n]` applies to field with name `errorElements[n]`.

#### properties

- `fields`: form definition data (see above). This property is required.
- `inlineErrors`: boolean value; if `true` will show validation error messages next to each field. If falsey or missing, validation errors will show in a block on their own af the bottom of the form.

#### Event handling

- `onProgress`: optional `function(ratio:number)` function that gets called every time the form is updated, reporting to its owner the fraction of metered components that pass validation as ratio of the total number of metered components. To convert this to percentages, simply compute `ratio * 100`.
- `onUpdate`: optional `function onUpdate(event, name, field, value)` where event is the change event for a form field element, `field` is the field object from the form definition, and `value` is the updated value for that field.

### `MultiPageForm`

Documentation pending, the code for this is still in flux

### `MultiSectionedForm`

Documentation pending, the code for this is still in flux
