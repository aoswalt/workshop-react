# Reusable Form

By accounting for optional initial values and providing a function for submission, we can build a form that can be reused for creation and updating.

First, let's create a component for a form field that renders a label/field pair and calls an `onChange` with the key and value together. We will come back to the reason for combining the key and value for the `onChange`.

```jsx
class Field extends React.Component {
  handleChange = ({ target: { value } }) =>
    this.props.onChange({ [this.props.field]: value })

  render() {
    const { label, type } = this.props
    const value = this.props.value || ''

    return (
    <div>
      <span>{label}</span>
      <input type={type} onChange={this.handleChange} value={value} />
    </div>)
  }
}
```

Using our `Field` component, we can easily build our form.

First, we set state based on an initial `contact` prop. To accomodate this, we also set a `defaultProp` of an initial `contact` entry. By doing so, we can work with an expected shape from the prop. The fields will have values if provided and the specified default values if not.

By having the `Field` component return an object with its field name as a key and updated value, the `onChange` function for the fields can simply pass the data directly to `this.setState`.

On the form's submission, the entire form state is simply passed to the `onSubmit` prop.

```jsx
class ContactForm extends React.Component {
  state = { ...this.props.contact }

  handleFieldChange = entry => this.setState(entry)

  onSubmit = e => {
    e.preventDefault()
    this.props.onSubmit(this.state)
  }

  render() {
    const { name, phone, address } = this.state

    return (
      <form onSubmit={this.onSubmit}>
        <Field
          label="Name"
          field="name"
          value={name}
          type="text"
          onChange={this.handleFieldChange}
        />
        <Field
          label="Phone"
          field="phone"
          value={phone}
          type="tel"
          onChange={this.handleFieldChange}
        />
        <Field
          label="Address"
          field="address"
          value={address}
          type="text"
          onChange={this.handleFieldChange}
        />
        <button type="submit">Submit</button>
      </form>
    )
  }
}

ContactForm.defaultProps = {
  contact: {
    name: '',
    phone: '',
    address: '',
  },
}
```

By separating the result of submitting the form, we can implement a create form by passing in the create function:

```jsx
<ContactForm onSubmit={postNewContact} />
```

Or, we can implement an edit form by passing in initial values and the edit function:

```jsx
<ContactForm onSubmit={updateContact} contact={existingContact} />
```
