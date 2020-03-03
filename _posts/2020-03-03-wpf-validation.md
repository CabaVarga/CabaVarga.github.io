---
layout: post
title: "WPF Validation techniques"
category: howto
date: 2020-03-03
---

You can choose between multiple techniques when you need to validate input in WPF. Every approach has its pros and cons plus you also need to consider the ease of integration into an MVVM based project.

The finest overview can be found at [Data validation in WPF](https://blog.magnusmontin.net/2013/08/26/data-validation-in-wpf/). This little article will be my attempt to summarize what I have collected from various sources. Another good starting point is the [Data validation](https://docs.microsoft.com/en-us/dotnet/desktop-wpf/data/data-binding-overview#data-validation) section of the documentation.  

## Summary

You have 3 basic techniques at your disposal, with the first one having two methods of implementation, differing by the place where your validation logic will live. The fourth category is custom validation, basically anything outside the framework provided infrastructure belongs there.

1. Validation Rule 
    1. ExceptionValidationRule 
    2. IDataErrorInfo
2. INotifyDataErrorInfo
3. Data annotations 
4. Custom techniques 

## ValidationRule

When using the WPF data binding model, you can associate [ValidationRules](https://docs.microsoft.com/en-us/dotnet/api/system.windows.data.binding.validationrules) (plural, collection) with your Binding or MultiBinding object. You can create custom rules by deriving from the [`ValidationRule`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.validationrule) class and implementing the `Validate` method. Optionally, use the built-in `ExceptionValidationRule`, which catches exceptions that are thrown during source updates, or the `DataErrorValidationRule`, which checks for errors raised by the `IDataErrorInfo` implementation of the source object. Examples: [How to: Implement Binding Validation](https://docs.microsoft.com/en-us/dotnet/framework/wpf/data/how-to-implement-binding-validation). 

TODO: Structure of the custom validation rule, return type, validationStep enumeration, validation process, **ValidationResult**, how to assign the rule to your binding (in code and in xaml), the role of the **ErrorTemplate**, where and how to check for validation errors (for example to disable the submit button). All of these are the basic concepts that you need to grasp in order to use to more complex validation scenarios.

### ExceptionValidationRule

Represents a rule that checks for exceptions that are thrown during the update of the binding source property. [Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.exceptionvalidationrule)

QUIRK: While testing the example from MSDN it occured to me that when the source type is decimal the textbox will not accept . (dot) and , (comma) keystrokes. Explanation [here](https://stackoverflow.com/questions/23228403/textbox-wont-allow-me-to-enter-in-a-decimal). In short: 1. does not make sense. You can switch off that behaviour by setting 

```csharp
System.Windows.FrameworkCompatibilityPreferences.KeepTextBoxDisplaySynchronizedWithTextProperty = false;
``` 

in your `App` constructor. 

QUIRK: The red border that will show up even if `ValidatesOnException` is set to `false` and there is no explicit `ExceptionValidationRule` set is due to the change introduced in .Net 4.0, based on [this comment](https://stackoverflow.com/a/19832594/4486196). The importance of the validation process procedure shows up here. Data conversion (`IValueConverter`) comes before validation, and if that step is unsuccessful the input field will turn red. Now, we need to check if that will automatically invalidate the state. 

```xml
<TextBox>
  <TextBox.Text>
    <Binding Path="Source" UpdateSourceTrigger="PropertyChanged">
      <Binding.ValidationRules>
        <ExceptionValidationRule />
      </Binding.ValidationRules>
    </Binding>
  </TextBox.Text>
</TextBox>
``` 

Setting the `ValidatesOnExceptions` property of the control to `true` has the same effect as adding the `ExceptionValidationRule` explicitly, based on [the docs](https://docs.microsoft.com/en-us/dotnet/api/system.windows.data.binding.validatesonexceptions). **Important:** if an exception is thrown, the binding engine creates a [ValidationError](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.validationerror) with the exception and adds it to the [Validation.Errors](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.validation.errors) collection of the bound element.  

### IDataErrorInfo



## INotifyDataErrorInfo

**SWEET SPOT** The most MVVM friendly solution, because you have complete insight into the process from your ViewModel. Also the most powerful, supporting complex validation scenarios.

The interface to implement:

```csharp
public interface INotifyDataErrorInfo
{
    bool HasErrors { get; }
    event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
    IEnumerable GetErrors(string propertyName);
}
``` 

The [official documentation](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifydataerrorinfo) is not helpful for the implementation, alas. 

To learn from: [The starting point](https://blog.magnusmontin.net/2013/08/26/data-validation-in-wpf/), [simple implementation, **notice property setter!**](https://kmatyaszek.github.io/wpf%20validation/2019/03/13/wpf-validation-using-inotifydataerrorinfo.html), the reference (and almost hidden) [INotifyDataErrorInfo](https://docs.microsoft.com/en-us/previous-versions/windows/silverlight/dotnet-windows-silverlight/ee652637(v=vs.95)?redirectedfrom=MSDN#syntax) Silverlight implementation (**TODO:** Copypasta for eternity), related [Data Binding in Silverlight](https://docs.microsoft.com/en-us/previous-versions/windows/silverlight/dotnet-windows-silverlight/cc278072%28v%3dvs.95%29#data-validation) with some good diagrams, clear explanations and nice visuals (to implement in my apps), and, finally, an [advanced implemenation](https://rehansaeed.com/model-view-viewmodel-mvvm-part4-inotifydataerrorinfo/) (with other goodies there). 

Little digression: you need to understand the role of the [Validation.Errors Attached Property](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.validation.errors).

## Data annotations



## Custom techniques

All the techniques mentioned above are meant to be used with data-binding. If you are not using data binding or if you do not want to add complexity into your project you are free to **handle** the validation in your *OnChange* and *OnSubmit* event handlers, inside your code-behind.