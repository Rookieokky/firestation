# firestation
A simple, configurable, realtime admin interface for Firebase, built on React.

See a live demo [here](https://s3-eu-west-1.amazonaws.com/firestation/demo/index.html) (and view the source [here](http://github.com/chrisvxd/firestation-demo)).

![Screenshot](https://s3-eu-west-1.amazonaws.com/firestation/Screenshot%202016-08-25%2017.43.59.png)

<a name="global-dependencies"></a>
## Global Dependencies

- npm
- webpack

<a name="installation"></a>
## Installation

Clone this repository.

Setup packages:

    npm i


<a name="example"></a>
## Example

You need to add a `firestation.config.js` file in the root of the project. This is where you configure your dashboard. Here's an example:

    import {ImageCell, TextCell, SelectCell} from 'components/cells.jsx';

    export default {
        auth: myAuthMethod,
        title: 'Pet Owners Database'
        refs: [
            {
                ref: myFirebaseRef.child('pets'),
                title: 'My Favorite Pets',
                orderBy: 'size.height',
                orderByDirection: 'desc',
                children: [
                    {
                        key: 'picture',
                        title: 'Profile Pic',
                        cell: ImageCell,
                        cellProps: {
                            width: '120',
                            height: '120'
                        },
                        canFilter: false
                    },
                    {
                        key: '_key',
                        cell: TextCell
                    },
                    {
                        key: 'name',
                        cell: TextCell
                    },                    {
                        key: 'size.height',
                        title: 'Height',
                        cell: TextCell
                    },
                    {
                        key: 'size.weight',
                        title: 'Weight',
                        cell: TextCell
                    },
                    {
                        key: 'species',
                        cell: SelectCell,
                        cellProps: {
                            options: [
                                {
                                    value: 'cat',
                                    title: 'Cat'
                                },
                                {
                                    value: 'dog',
                                    title: 'Dog'
                                }
                            ]
                        }
                    }
                ]
            },
            {
                ref: myFirebaseRef.child('owners').orderByChild('lazy').equalTo(true) // Full ref chaining support
                title: 'Lazy Owners',
                resolve: function (key, val, callback) {  // Custom firebase resolve method
                    val.calculatedLaziness = Math.random();
                    callback(val);
                },
                children: [
                    {
                        key: 'surname',
                        cell: TextCell,
                        canWrite: true
                    },
                    {
                        key: 'lazy',
                        cell: TextCell
                    },
                    {
                        key: 'calculatedLaziness',
                        cell: TextCell
                    }
                ]
            }
        ]
    }

That's it. The `ref` array contains configuration for each firebase ref you want to render in the dashboard, and each of the objects in `children` represent a column that will be rendered for a key on _that ref_, and defines how to render them.

<a name="configuration-api"></a>
## Configuration API
`firestation.config.js` contains the configuration for your firestation. It's important to do your exports using `ES6`, as shown in the [example above](#example). You'll need to export an object containing your configuration.

Firestation works by rendering a table for each `ref` you're interested in, with each column representing a `key` for the objects in that `ref`.

<a name="top-level"></a>
### Top-level
The top level defines how to connect to your firebase:

- `auth` (function) - an authentication method to authenticate with your firebase server. It's up to you to implement that, but is should take a `callback`.
- `title` (string) - the title of your database. This will render in the top left.
- `refs` (array) - an array of `ref` configuration that will describe how to render your firebase. [See Refs](#refs) for more.

Example:

    var myFirebaseRef = ...;

    // Custom authentication method. Up to you to implement.
    var myAuthMethod = function (callback) {
      ...
      callback();  
    };

    export default {
      auth: myAuthMethod,
      refs: [...]
    }

<a name="refs"></a>
### Refs
Each item in the top-level `refs` renders to a table. It defines a [firebase ref](https://www.firebase.com/docs/web/guide/understanding-data.html#section-creating-references) for your database, and configuration for how it should be rendered:

- `ref` (Firebase) - a full firebase reference for the child you want to create a table for. Supports full chaining, such as `orderByChild`.
- `children` (array) - an array of configurations that define how each key should be rendered. See [Cell API](#cell-api) for cell configuration.
- `resolve` (function, optional) - a custom method for running custom resolves before rendering the item ([see Resolves](#resolves))
- `title` (string, optional) - human-readable title for this ref configuration
- `orderBy` (string, optional) - the default `key` to order your
- `orderByDirection` (string, optional) - the direction (`asc` or `desc`) for the ordering of your `orderBy` value
- `rangeStart` (integer, optional) - the initial index of the first item in rendered range. Defaults to `1`
- `rangeEnd` (integer, optional) - the initial index of the last of the rendered range. Defaults to `10`

Example:

    {
        ref: myFirebaseRef.child('pets'),
        title: 'Pets',
        orderBy: 'age',
        orderByDirection: 'asc',
        resolve: function (value, callback) {
            value.myExtraField = 'Wohoo!';
            callback(value);
        },
        rangeStart: 1,
        rangeEnd: 25,
        children: [
            ...
        ]
    }

<a name="cell-api"></a>
## Cell API

Cells render your firebase values depending on their type and how you want to display them. For example, there's an `ImageCell` which loads an image from a URL, or just a lowly `TextCell`. Many of them have both read and write states, making them extremely powerful for managing your data. They are all built on [`React`](http://facebook.github.io/react/), which makes [defining custom cells](#custom-cells) extremely easily.

Each configuration takes the following properties:

- `key` (string) - The firebase key for this column
- `cell` (object) - The cell to render the value with (see below)
- `cellProps` (object) - Cell-specific attributes which are passed to the cell
- `title` (string) - A human-readable name for this key
- `canFilter` (boolean, optional) - Whether this column can be filtered using column searching
- `canWrite` (boolean, optional) - Whether the cells can be used in write-mode (currently only partial support)
- `width` (string or integer, optional) - Width of the cell column ([passed directly to `<th>`](http://www.w3schools.com/tags/att_col_width.asp))

`key` can also take some special values, indiciated by a `_` prefix:

- `_key` - The key (id) for this object

Firestation provides various cells for common use cases:

- [`TextCell`](#text-cell)
- [`LongTextCell`](#long-text-cell)
- [`NumberCell`](#number-cell)
- [`BooleanCell`](#boolean-cell)
- [`ImageCell`](#image-cell)
- [`SelectCell`](#select-cell)
- [`TimeSinceCell`](#time-since-cell)
- [`DateCell`](#date-cell)
- [`CurrencyCell`](#currency-cell)
- [`ButtonCell`](#button-cell)
- [`DropdownCell`](#dropdown-cell)

We're adding to this (see [Future Cells](#future-cells)), but [you can write custom react cells](#custom-cells) if you need anything fancy.

<a name="text-cell"></a>
### TextCell
Renders the value as simple text. Does not require any `cellProps`.

This cell _does_ support the `canWrite` method.

<a name="long-text-cell"></a>
### LongTextCell
Renders the value as text in a mulitline textarea. Does not require any `cellProps`.

This cell _does_ support the `canWrite` method.

<a name="number-cell"></a>
### Number
Renders the value as number. Does not require any `cellProps`.

This cell _does_ support the `canWrite` method.

<a name="boolean-cell"></a>
### BooleanCell
Renders a boolean value as a checkbox. Takes the following `cellProps`:

- `label` (string, optional) - inline label for the checkbox

Example:

    {
        key: 'likesChicken',
        cell: BooleanCell,
        cellProps: {
            label: 'Chicken Lover?'
        }
    }

This cell _does_ support the `canWrite` method.

<a name="image-cell"></a>
### ImageCell
Renders the value as an image. Takes the following `cellProps`:

- `width` (string) - width of the image
- `height` (string) - height of the image

Example:

    {
        key: 'species',
        cell: SelectCell,
        cellProps: {
            width: '50',
            height: '50'
        }
    }

This cell __does not__ support the `canWrite` method.

<a name="select-cell"></a>
### SelectCell
Renders value as an option in a list of options, with human-readable counterparts. Takes the following `cellProps`:

- `options` (array) - contains objects with the `key` and `title` for each possible value for this key

Example:

    {
        key: 'species',
        cell: SelectCell,
        cellProps: {
            options: [
                {
                    value: 'cat',
                    title: 'Cat'
                },
                {
                    value: 'dog',
                    title: 'Dog'
                }
            ]
        }
    }

This cell _does_ support the `canWrite` method.

<a name="date-cell"></a>
### DateCell
Renders a date in seconds or ISO format to a human-readable date. Write mode uses [a datepicker](https://github.com/YouCanBookMe/react-datetime) to make date selection easy. Formats for your locale and handles daylight changes.

- `dateFormat` (string) - date format. Defaults to UK format, i.e. `DD/MM/YY`. Follows the [moment.js format](http://momentjs.com/docs/#/displaying/format/)
- `timeFormat` (string) - time format. Also follows the [moment.js format](http://momentjs.com/docs/#/displaying/format/)
- `saveFormat` (string) - the format to save to firebase. Either `numeric` (in ms) or `iso`. Defaults to `iso`.

Example:

    {
        key: 'startAt',
        cell: Start,
        cellProps: {
            dateFormat: 'MM/DD/YY',
            saveFormat: 'numeric'
        },
        canWrite: true
    }

This cell _does_ support the `canWrite` method.

<a name="time-since-cell"></a>
### TimeSinceCell
Renders a date in seconds or ISO format to a time since string, for example "2 days ago".

This cell __does not__ support the `canWrite` method.

<a name="currency-cell"></a>
### CurrencyCell
Renders a number as a currency. Takes the following `cellProps`:

- `symbol` (string) - object containing the `key` and `title` of each option

Support for adding trailing zeros will be added.

Example:

    {
        key: 'price',
        cell: CurrencyCell,
        cellProps: {
            symbol: '£'
        }
    }

This cell __does not__ support the `canWrite` method.

<a name="button-cell"></a>
### ButtonCell
Renders a button. Takes the following `cellProps`:

- `title` (string) - title of the button
- `type` (string) - type of the button. Options are `primary`, `success`, `warning` and `danger`. Defaults to `primary`
- `disabled` (string) - whether the cell is disabled. Defaults to `false`
- `action` (function) - method to call when the button is clicked. You can modify `title`, `type` and `disabled` in the callback.

Simple example:

    {
        key: '',
        cell: ButtonCell,
        cellProps: {
            title: 'Click me!',
            action: function (rowKey, rowValue, callback) {
                console.log('Woohoo! Performed action on', rowKey);

                callback({
                  disabled: true
                });
            }
        }
    }

<a name="dropdown-cell"></a>
### DropdownCell
Renders a dropdown that can trigger actions. Takes the following `cellProps`:

- `title` (string) - title of the button
- `disabled` (string) - whether the cell is disabled. Defaults to `false`
- `items` (function) - an array of items to render, made up of `actions`, `dividers` and `headers`. See example below for usage.

Simple example:

    {
        key: '',
        cell: DropdownCell,
        cellProps: {
            title: 'Drop it like it's hot',
            items: [
                {
                    label: 'Action',
                    action: function (rowKey, rowValue) {
                        console.log('Bam!', rowKey, rowValue);
                    }
                },
                { label: 'Another action' },
                { label: 'Something else here' },
                { type: 'divider' },
                { type: 'header', label: 'Dropdown header' },
                { label: 'Separated link' }
            ]
        }
    }

<a name="future-cells"></a>
### Future Cells

Cells planned but not yet implemented:

No cells are currently planned at this time.

<a name="custom-cells"></a>
### Custom Cells
All cells are [built on react](http://facebook.github.io/react/). They are totally pluggable, so it is possible to write custom cells. It is **not** necessary to have an extensive knowledge of React to implement them. Here is a basic, read-only cell that renders an image:

    import React from 'react';

    var FancyImageCell = React.createClass({
        render: function () {
            return (
                <img
                    src={this.props.value}
                    width={this.props.extras.width}
                    height={this.props.extras.height}>
                </img>
            );
        }
    });

This is all basic React. The properties (`this.props`) that the cell will from firestation receive are:

- `value` - the value for the key this cell will render
- `clean` (bool) - whether the value of this cell is in sync with firebase. Useful for handling write state.
- `rowKey` - the value for the entire row
- `rowValue` - the key for entire row
- `extras` (object) - the [`cellProps`](#cell-api) for that configuration. These could be anything you need.
- `canWrite` (bool) - the [`canWrite`](#cell-api) for that configuration, determining if the cell should be writable
- `childKey` (string) - the `key` that this cell is rendering (e.g. `age`)
- `valueChanged` (function) - a method to inform firestation that the value has changed, enabling the Save Button which will write the changes to firebase.

Here's a more advanced example, implementing a read-and-writable cell that renders a text input:

    import React from 'react';

    var FancyTextCell = React.createClass({
        getInitialState: function () {
            return {
                value: this.props.value
            }
        },
        handleChange: function (event) {
            this.setState({
                value: event.target.value
            });
            this.props.valueChanged(this.props.childKey, event.target.value);
        },
        render: function () {
            // We're in sync with firebase, so ensure values of props and state match.
            if (this.props.clean) {
                this.state.value = this.props.value;
            }

            if (this.props.canWrite) {
                // Read and write
                return (
                    <input type="text" value={this.state.value} onChange={this.handleChange}></input>
                )
            } else {
                // Read only
                return (
                    <input type="text" value={this.props.value}></input>
                )
            }
        }
    });

You can do whatever you want with the functionality of these cells, easily tailoring firestation to your specific needs.

<a name="resolves"></a>
## Resolves
Resolve methods allow you to modify the `value` for each Firebase Snapshot _before_ it gets rendered in the table. This greatly extends the capability of firestation, enabling you to do things calculate values or even run extra firebase calls to flatten related objects!

Calculation Example - adding a `yearBorn` value when we only know the age:

    function (key, value, callback) {
        var currentYear = new Date().getFullYear();
        value.yearBorn = currentYear - value.age;
        callback(value);
    }

Flattening (denormalizing) related objects Example - `owner` onto `pet` using the `owner` key:

    function (key, petValue, callback) {
        myFirebaseRef.child('owners').child(petValue.ownerKey).once('value', function (ownerSnapshot) {
            petValue.owner = ownerSnapshot.value();
            callback(petValue);
        });
    }

Neat, huh?

A small caveat with this method for flattening resources is that firestation will not monitor for or render changes in the flattened object, only the main resource. You could use `.on('child_changed')` or similar method to address this, but this is not officially supported or tested and firestation may behave unexpectedly.

__Please note, since these values are calculated and do not actually exist on the firebase resource, they are intrinsically read-only. It is impossible to write to them.__

<a name="favicons"></a>
## Favicons
You can add favicons to your project. `index.html` will render favicons rendered by [RealFaviconGenerator](http://realfavicongenerator.net), placed in the `favicons` directory of your project (probably in `dist`).

<a name="running-locally"></a>
## Running locally

Just use

    npm start

Now go to http://127.0.0.1:8080.

<a name="building-for-distribution"></a>
## Building for distribution

Build that shiz:

    webpack

Deploy `dist` wherever.
