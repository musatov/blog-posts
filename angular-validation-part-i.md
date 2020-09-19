---
title: Angular forms validation. Part I. Single control validation.
description: At this series of posts, I am going through a few Reactive Forms Validation samples from the simplest one to more advance. At the first part of the series, we are going to talk about single form control validation
date: 2020-01-17
---

## Angular forms validation. Part I. Single control validation.

Angular provided with the powerful tools for managing forms. At this series of posts, I am going through a few _Reactive Forms Validation_ samples from the simplest one to more advance.

At the first part of the series, we are going to talk about _single_ form control validation. Zero or any number of synchronous and asynchronous validators can be applied to a single control.

### Let's start with the simplest possible example

FormControl with no validation and some angular component markup to show this FormControl.

```typescript
export class SingleControlComponent {
  readonly emailControl = new FormControl();
}
```

```html
<label for="email">Email</label>
<input name="email" type="text" [formControl]="emailControl" />
```

Running this sample we gonna see similar result at the browser
![Sample control](https://thepracticaldev.s3.amazonaws.com/i/2gxmi03rmharha5flzeu.png)

### Add some synchronous validation to the FormControl.

We are going to use [validators](https://angular.io/api/forms/Validators) provided by the Angular framework. `Validators.required` - requires to have a non-empty value. `Validators.email` - checking email against the pattern which is based on the definition of a valid email address in the [WHATWG HTML specification](https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address) with some enhancements to incorporate more RFC rules.

```typescript
export class SingleControlComponent {
  readonly emailControl = new FormControl(null, [
    Validators.required,
    Validators.email
  ]);
}
```

As the next step we need handle validation results at the control markup. FormControl API is helpfull here. `hasError` method checks the presence of error code ('required', 'email', etc.) at the errors collection of the control.

```html
<label for="email">Email</label>
<input name="email" type="text" [formControl]="emailControl" />
<div class="errors">
  <span *ngIf="emailControl.hasError('required')">Email is Required</span>
  <span *ngIf="emailControl.hasError('email')">Email is mailformed</span>
</div>
```

After this changes our control should work like at the following record
![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/e2wy76tecui2ygj610fo.gif)

### Implement asynchronous validator.

The next step is going to be the implementation of some asynchronous validation. Let's create some async validator pretend to check email presence at the server.

```typescript
function emailMustNotBeUsed(
  control: AbstractControl
): Observable<ValidationErrors | null> {
  return control.value === 'used@email.com'
    ? of({ 'email-is-used': 'Email was used' }).pipe(delay(2000))
    : of(null).pipe(delay(2000));
}
```

We are delaying execution result to simulate a call to some remote resource. And now we are appening the async validator to the FromControl

```typescript
export class SingleControlComponent {
  readonly emailControl = new FormControl(
    null,
    // Sync validators
    [Validators.required, Validators.email],
    // Async validator
    emailMustNotBeUsed
  );
}
```

Handling vlidators result at the component markup by adding the following element

```html
...
<span *ngIf="emailControl.hasError('email-is-used')"
  >{{emailControl.getError('email-is-used')}}</span
>
...
```

_Note: Error message from the validator is shown at the UI by using `getError` method. It can be helpfull if async validator return specific error message (received from the server for example)._

Results of all our changes are gonna be like at the record belove
![The Results](https://thepracticaldev.s3.amazonaws.com/i/blhzn7h0oflvn603jhwd.gif)

### Validators execution order

The order of execution is guaranteed. At first, all sync validators are executed in order of declaration. As soon sync validators returned results and there are no errors, async validators are started. Async validators are _started_ in order of declaration, but execution results order is not guaranteed. Let's take a look at the code.

```typescript
export class SingleControlComponent {
  readonly emailControl = new FormControl(
    null,
    // Sync validators
    [Validators.required, Validators.email],
    // Async validators
    [emailMustNotBeUsed, emailIsForbidden]
  );
}
```

The implementation of async validators used for this form control is present belove

```typescript
function emailMustNotBeUsed(
  control: AbstractControl
): Observable<ValidationErrors | null> {
  console.log('First async validator stars executing');
  const resutl$ =
    control.value === 'used@email.com'
      ? of({ 'email-is-used': 'Email was used' }).pipe(delay(2000))
      : of(null).pipe(delay(2000));
  return resutl$.pipe(
    finalize(() => console.log('First async validator completed'))
  );
}

function emailIsForbidden(
  control: AbstractControl
): Observable<ValidationErrors | null> {
  console.log('Second async validator stars executing');
  const resutl$ =
    control.value === 'forbidden@email.com'
      ? of({ 'email-is-forbidden': 'Email was forbidden' }).pipe(delay(1500))
      : of(null).pipe(delay(1500));

  return resutl$.pipe(
    finalize(() => console.log('Second async validator completed'))
  );
}
```

The console output of running this sample is going to be exactly this:

> First async validator stars executing
> Second async validator stars executing
> Second async validator completed
> First async validator completed

First two lines are gonna be printed the same order no matter of internal async validators implementation. Order of the other two is dependant on async validation delay.

### Conclusion

Thanks for reading. I hope it helped you in some way. All code samples are available at the [Github](https://github.com/musatov/angular-samples/tree/master/form-validators).

There are two other articles available on the topic:
[Angular forms validation. Part II. FormGroup validation.](https://dev.to/musatov/angular-forms-validation-part-ii-formgroup-validation-3g68)
[Angular forms validation. Part III. Async Validators gotchas.](https://dev.to/musatov/angular-forms-validation-part-iii-async-validators-gotchas-5c0o)
