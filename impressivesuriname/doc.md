# NB CPF Frontend Assets and Country/Phone Field Integration

## Plugin file
wp-content\plugins\country-phone-field-contact-form-7\includes\include-js-css.php

This file registers and initializes frontend CSS/JavaScript assets for a
WordPress plugin/theme feature that adds country selector and international phone
number behavior to Contact Form 7 fields.

It uses:

- WordPress enqueue APIs
- jQuery
- `intlTelInput`
- `countrySelect`
- Contact Form 7 field classes:
  - `.wpcf7-countrytext`
  - `.wpcf7-phonetext`

---

## Main Purpose

The code does the following:

1. Loads required CSS and JavaScript files for country and phone inputs.
2. Initializes country dropdowns on `.wpcf7-countrytext` fields.
3. Initializes international telephone inputs on `.wpcf7-phonetext` fields.
4. Supports default, preferred, included, and excluded countries.
5. Optionally auto-detects the visitor's country using an IP geolocation API.
6. Stores the selected phone country dial code into hidden form fields.
7. Adds a searchable country dropdown for phone fields.
8. Defines an optional AJAX helper for country detection, although its AJAX hooks
   are currently commented out.

---

## WordPress Hook

```php
add_action( 'wp_enqueue_scripts', 'nb_cpf_embedCssJs' );
```

The function `nb_cpf_embedCssJs()` runs on the frontend when WordPress enqueues
scripts and styles.

---

## Function: `nb_cpf_embedCssJs()`

### Purpose

This is the main function in the file. It loads frontend assets and injects
inline JavaScript used to initialize country and phone fields.

---

## Enqueued Styles

```php
wp_enqueue_style(
    'nbcpf-intlTelInput-style',
    NB_CPF_URL . 'assets/css/intlTelInput.min.css'
);

wp_enqueue_style(
    'nbcpf-countryFlag-style',
    NB_CPF_URL . 'assets/css/countrySelect.min.css'
);
```

### What They Do

- `intlTelInput.min.css` styles the international phone input.
- `countrySelect.min.css` styles the country selector input.

---

## Enqueued Scripts

```php
wp_enqueue_script(
    'nbcpf-intlTelInput-script',
    NB_CPF_URL . 'assets/js/intlTelInput.min.js',
    array( 'jquery' ),
    false,
    true
);

wp_enqueue_script(
    'nbcpf-countryFlag-script',
    NB_CPF_URL . 'assets/js/countrySelect.min.js',
    array( 'jquery' ),
    false,
    true
);
```

### What They Do

- `intlTelInput.min.js` adds international telephone input functionality.
- `countrySelect.min.js` adds country selection functionality.
- Both depend on jQuery.
- Both are loaded in the footer because the final argument is `true`.

---

## Inline CSS for Country Search

The code adds inline CSS to style a custom search box inside the phone country
dropdown.

```php
wp_add_inline_style( 'nbcpf-intlTelInput-style', '...' );
```

### Added CSS Classes

| Class | Purpose |
|---|---|
| `.nbcpf-country-search-item` | Wrapper list item for the search input |
| `.nbcpf-country-search` | Search input inside the dropdown |
| `.nbcpf-country-search-no-results` | Message shown when no country matches |

---

## Localized JavaScript Object

```php
wp_localize_script(
    'nbcpf-countryFlag-script',
    'nbcpf',
    array(
        'ajaxurl' => site_url() . '/wp-admin/admin-ajax.php',
    )
);
```

This exposes a JavaScript object named `nbcpf` with the WordPress AJAX URL.

Example in JavaScript:

```js
nbcpf.ajaxurl
```

This could be used for AJAX requests to WordPress.

---

## Plugin Settings

The code reads saved options from WordPress:

```php
$nb_cpf_settings_options = get_option( 'nb_cpf_options' );
```

These settings control the behavior of the country and phone fields.

---

## Country Field Options

The following options affect `.wpcf7-countrytext` fields.

### `defaultCountry`

```php
$nb_cpf_settings_options['defaultCountry']
```

If set, the country selector starts with this country.

Example generated JavaScript:

```js
defaultCountry: "us",
```

---

### `onlyCountries`

```php
$nb_cpf_settings_options['onlyCountries']
```

Limits the country dropdown to only the listed countries.

Example option value:

```text
us,ca,br
```

Generated JavaScript:

```js
onlyCountries: ["us", "ca", "br"],
```

---

### `preferredCountries`

```php
$nb_cpf_settings_options['preferredCountries']
```

Shows selected countries at the top of the dropdown.

Example:

```js
preferredCountries: ["us", "br"],
```

---

### `excludeCountries`

```php
$nb_cpf_settings_options['excludeCountries']
```

Removes selected countries from the dropdown.

Example:

```js
excludeCountries: ["ru", "kp"],
```

---

## Phone Field Options

The following options affect `.wpcf7-phonetext` fields.

### `phone_defaultCountry`

```php
$nb_cpf_settings_options['phone_defaultCountry']
```

Sets the initial country for the phone input.

Generated JavaScript example:

```js
initialCountry: "br",
```

---

### `phone_onlyCountries`

Limits phone input country list to specific countries.

```js
onlyCountries: ["us", "ca", "br"],
```

---

### `phone_preferredCountries`

Shows preferred phone countries at the top of the phone dropdown.

```js
preferredCountries: ["us", "br"],
```

---

### `phone_excludeCountries`

Excludes countries from the phone dropdown.

```js
excludeCountries: ["ru", "kp"],
```

---

### `phone_nationalMode`

Controls whether phone numbers are entered in national or international format.

```php
if (
    isset( $nb_cpf_settings_options['phone_nationalMode'] ) &&
    $nb_cpf_settings_options['phone_nationalMode'] == 1
) {
    $phone_nationalMode = 'true';
} else {
    $phone_nationalMode = 'false';
}
```

Generated JavaScript:

```js
nationalMode: true
```

or:

```js
nationalMode: false
```

When `nationalMode` is disabled, the script automatically prefixes the phone
number with the selected dial code.

---

## Auto Country Detection

The code supports automatic country detection when either of these settings is
enabled:

```php
$nb_cpf_settings_options['country_auto_select']
$nb_cpf_settings_options['phone_auto_select']
```

If either setting is enabled, the frontend JavaScript sends a request to:

```text
https://ipinfo.io/json
```

The API response is expected to include a `country` value.

Example response:

```json
{
  "country": "BR"
}
```

If the country is found, the script uses it to initialize the country and/or
phone fields.

Example generated JavaScript:

```js
defaultCountry: response.country.toLowerCase()
```

For phone fields:

```js
initialCountry: response.country.toLowerCase()
```

If the API request fails, the code falls back to the default settings by calling:

```js
render_country_flags();
```

---

## JavaScript Initialization

The generated inline JavaScript is wrapped in an immediately invoked function
expression:

```js
(function($) {
  $(function() {
    // Code runs when the DOM is ready.
  });
})(jQuery);
```

This ensures:

- jQuery is available as `$`.
- The code runs after the page DOM has loaded.
- The global scope is not polluted.

---

## Country Field Initialization

The country fields are initialized with:

```js
$(".wpcf7-countrytext").countrySelect({
  defaultCountry: "br",
  onlyCountries: ["br", "us"],
  preferredCountries: ["br"],
  excludeCountries: []
});
```

Actual options depend on the saved WordPress plugin settings.

---

## Phone Field Initialization

The phone fields are initialized with:

```js
$(".wpcf7-phonetext").intlTelInput({
  autoHideDialCode: true,
  autoPlaceholder: true,
  nationalMode: false,
  separateDialCode: true,
  hiddenInput: "full_number",
  initialCountry: "br"
});
```

### Important Phone Options

| Option | Description |
|---|---|
| `autoHideDialCode` | Automatically hides dial code when appropriate |
| `autoPlaceholder` | Adds placeholder based on selected country |
| `nationalMode` | Controls national/international number format |
| `separateDialCode` | Shows dial code separately beside the input |
| `hiddenInput` | Creates/stores a hidden full phone number field |
| `initialCountry` | Sets the default selected phone country |

---

## Hidden Country Code Input

For every `.wpcf7-phonetext` field, the script finds the selected dial code:

```js
var dial_code = $(this)
  .siblings(".flag-container")
  .find(".selected-flag .selected-dial-code")
  .text();
```

It then stores the dial code in a hidden input field.

The hidden input name is expected to follow this pattern:

```text
{phone-field-name}-country-code
```

Example:

If the phone field name is:

```text
phone
```

The hidden input should be:

```text
phone-country-code
```

The script updates it like this:

```js
$("input[name=" + hiddenInput + "-country-code]").val(dial_code);
```

---

## Phone Country Change Event

When the user changes the selected phone country, this event runs:

```js
$(".wpcf7-phonetext").on("countrychange", function() {
  var dial_code = $(this)
    .siblings(".flag-container")
    .find(".selected-flag .selected-dial-code")
    .text();

  var hiddenInput = $(this).attr("name");

  $("input[name=" + hiddenInput + "-country-code]").val(dial_code);
});
```

### Purpose

It keeps the hidden country code field synchronized with the selected country.

---

## Automatic Dial Code Prefix

When `phone_nationalMode` is not enabled, the script listens for keyup events on
phone fields.

```js
$(".wpcf7-phonetext").on("keyup", function() {
  var dial_code = $(this)
    .siblings(".flag-container")
    .find(".selected-flag .selected-dial-code")
    .text();

  var value = $(this).val();

  if (value == "+") {
    $(this).val("");
  } else if (value.indexOf("+") == "-1") {
    $(this).val(dial_code + value);
  } else if (value.indexOf("+") > 0) {
    $(this).val(dial_code + value.substring(dial_code.length));
  }
});
```

### What It Does

- If the user types only `+`, it clears the field.
- If the value does not include `+`, it prefixes the selected dial code.
- If `+` appears in the wrong position, it attempts to normalize the number.

Example:

Selected country dial code:

```text
+55
```

User types:

```text
11999999999
```

The field becomes:

```text
+5511999999999
```

---

## Country Text Keyup Behavior

The country field also listens for keyup events:

```js
$(".wpcf7-countrytext").on("keyup", function() {
  var country_name = $(this)
    .siblings(".flag-dropdown")
    .find(".country-list li.active span.country-name")
    .text();

  if (country_name == "") {
    var country_name = $(this)
      .siblings(".flag-dropdown")
      .find(".country-list li.highlight span.country-name")
      .text();
  }

  var value = $(this).val();

  $(this).val(country_name + value.substring(country_name.length));
});
```

### Purpose

This tries to keep the input value aligned with the currently selected or
highlighted country name.

---

## Custom Search in Phone Country Dropdown

The second inline script adds a search input to the `intlTelInput` country
dropdown.

---

### Search Setup Function

```js
function setupPhoneCountrySearch($dropdown) {
  var $countryList = $dropdown.find(".country-list").first();

  if (!$countryList.length || $countryList.data("nbcpfSearchReady")) {
    return;
  }

  $countryList.data("nbcpfSearchReady", true);

  $countryList.prepend(
    '<li class="' +
      searchItemClass +
      '">' +
      '<input class="' +
      searchInputClass +
      '" type="search" autocomplete="off" placeholder="Search country" aria-label="Search country" />' +
      "</li>"
  );

  $countryList.append(
    '<li class="' + noResultsClass + '">No countries found</li>'
  );
}
```

### What It Does

- Finds the phone dropdown country list.
- Prevents duplicate search inputs using `data("nbcpfSearchReady")`.
- Adds a search input at the top of the list.
- Adds a "No countries found" message at the bottom.

---

### Country Filter Function

```js
function filterPhoneCountries($input) {
  var query = $.trim($input.val()).toLowerCase();
  var $countryList = $input.closest(".country-list");
  var matches = 0;

  $countryList.children(".country").each(function() {
    var $country = $(this);

    var isMatch =
      !query ||
      $country.text().toLowerCase().indexOf(query) !== -1 ||
      String($country.data("country-code") || "")
        .toLowerCase()
        .indexOf(query) !== -1;

    $country.toggle(isMatch);

    if (isMatch) {
      matches++;
    }
  });

  $countryList.children(".divider").toggle(!query);
  $countryList.children("." + noResultsClass).toggle(query && matches === 0);
}
```

### What It Does

The search filters countries by:

- Country name text
- Country code stored in `data-country-code`

If no countries match, it shows:

```text
No countries found
```

---

### Search Input Activation

The search input is added when the user clicks the phone flag selector:

```js
$(document).on("click", ".intl-tel-input .selected-flag", function() {
  var $dropdown = $(this).closest(".intl-tel-input");

  setupPhoneCountrySearch($dropdown);

  window.setTimeout(function() {
    $dropdown
      .find("." + searchInputClass)
      .val("")
      .trigger("input")
      .focus();
  }, 0);
});
```

### What It Does

- Detects when the phone country dropdown is opened.
- Inserts the search field if it has not already been inserted.
- Clears the search.
- Focuses the search input.

---

### Search Event Handling

```js
$(document).on("input", "." + searchInputClass, function() {
  filterPhoneCountries($(this));
});
```

This filters countries as the user types.

---

### Preventing Dropdown Interference

```js
$(document).on(
  "click mousedown mouseup keydown keyup keypress",
  "." + searchInputClass,
  function(event) {
    event.stopPropagation();
  }
);
```

This prevents typing or clicking inside the search box from accidentally closing
or interacting incorrectly with the dropdown.

---

## Function: `nb_cpf_autoCountryDetection()`

### Purpose

This function is designed to perform server-side country detection using an IP
address.

However, the AJAX hooks are currently commented out:

```php
// add_action('wp_ajax_nopriv_auto_country_detection', 'nb_cpf_autoCountryDetection');
// add_action('wp_ajax_auto_country_detection', 'nb_cpf_autoCountryDetection' );
```

Because these hooks are commented out, this function is not currently available
through WordPress AJAX.

---

## Country Detection Flow

```php
$ip_address = $_REQUEST['ip'];
```

The function reads an IP address from the request.

If an IP address is provided, it calls:

```text
https://ipwho.is/{ip_address}
```

Example:

```text
https://ipwho.is/8.8.8.8
```

The request is made with:

```php
wp_safe_remote_get(
    $api_url,
    array(
        'timeout' => 3,
    )
);
```

The response body is retrieved:

```php
$response = wp_remote_retrieve_body( $response );
```

The JSON response is decoded:

```php
$parse_json = json_decode( $response, true );
```

Then returned as JSON:

```php
echo json_encode( $parse_json );
```

Finally, the function ends with:

```php
wp_die();
```

---

## Current Limitations and Notes

### 1. AJAX Function Is Disabled

The function `nb_cpf_autoCountryDetection()` will not run through WordPress AJAX
unless these hooks are uncommented:

```php
add_action(
    'wp_ajax_nopriv_auto_country_detection',
    'nb_cpf_autoCountryDetection'
);

add_action(
    'wp_ajax_auto_country_detection',
    'nb_cpf_autoCountryDetection'
);
```

---

### 2. External IP Lookup Uses `ipinfo.io`

The frontend auto-detection currently uses:

```text
https://ipinfo.io/json
```

This request happens from the visitor's browser.

Depending on the service limits or privacy requirements, this may need review.

---

### 3. Unused PHP Variable

The code stores the visitor IP address:

```php
$IPaddress = $_SERVER['REMOTE_ADDR'];
```

But this variable is not used anywhere else in the function.

---

### 4. Inline JavaScript Is Dynamically Built

The JavaScript is generated as a PHP string. This works, but it can become hard
to maintain as the logic grows.

A cleaner approach would be to:

1. Move most JavaScript into a separate `.js` file.
2. Pass WordPress settings using `wp_localize_script()` or
   `wp_add_inline_script()`.
3. Initialize fields using a structured JavaScript config object.

---

### 5. Input Sanitization Could Be Improved

Some settings are directly inserted into generated JavaScript. For better
security and stability, values should be sanitized and escaped before output.

Recommended WordPress functions include:

```php
sanitize_text_field()
esc_js()
wp_json_encode()
```

---

### 6. `$_REQUEST['ip']` Should Be Sanitized

In `nb_cpf_autoCountryDetection()`, the IP address is read directly:

```php
$ip_address = $_REQUEST['ip'];
```

A safer approach would be:

```php
$ip_address = isset( $_REQUEST['ip'] )
    ? sanitize_text_field( wp_unslash( $_REQUEST['ip'] ) )
    : '';
```

---

## Summary

This code integrates country and international phone number selection into
Contact Form 7 fields.

It:

- Loads country and phone selector libraries.
- Applies selectors to `.wpcf7-countrytext` and `.wpcf7-phonetext`.
- Supports configurable default, preferred, included, and excluded countries.
- Can auto-detect the visitor country using an external IP lookup service.
- Stores phone dial codes in hidden fields.
- Adds a custom search box to the phone country dropdown.
- Includes a server-side country detection helper, although it is currently not
  active because its AJAX hooks are commented out.
