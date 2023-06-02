# Plugin Installation and Activation Guide

This guide provides instructions for installing and activating the Preschool Registration Plugin, along with example GET requests and their expected return values.

## Installation

1. Download the plugin zip file from the GitHub repository.
2. Locate the root folder of your website and navigate to the `wp-content` directory.
3. Inside the `wp-content` directory, find the `plugins` folder and extract the contents of the downloaded zip file into this folder.

## Activation

1. Access your WordPress Dashboard and click on the "Plugins" tab.
2. Look for the newly added plugin named "Preschool Registration" in the list.
3. Click the "Activate" button next to the "Preschool Registration" plugin to activate it.

## Example GET requests

### Request 1: Get Preschool Details based on Post ID

- URL: `http://preschool-registration.local/wp-json/preschool-reg/v1/15`
- Method: GET

### Expected Response

```json
{
    "id":"15",
    "preschool_name":"Little Explorers",
    "preschool_address":"123 Main Street, Philadelphia, PA 19101",
    "registration_days":["Mon","Wed","Fri"],
    "registration_time_start":"11:00",
    "registration_time_end":"16:00",
    "accepting_registrations":"on"
}
```

### Request 2: Get Preschool Details based on **incorrect** Post ID

- URL: `http://preschool-registration.local/wp-json/preschool-reg/v1/20`
- Method: GET

### Expected Response

```json
{
    "code":"invalid_post_id",
    "message":"Invalid Post ID.",
    "data":
    {
        "status":400
    }
}
```

You can try for post IDs 15, 16, 17, 18, 19

### Request 3: Get Preschools based on registration date and time 

  - URL: `http://preschool-registration.local/wp-json/preschool-reg/v1/filter?registration_time=2023-06-02T13:00:00`
- Method: GET

### Expected Response

```json
[
    {
        "id":19,
        "preschool_name":"Little Sprouts",
        "preschool_address":"987 Pine Road, Philadelphia, PA 19105",
        "registration_days":["Mon","Wed","Fri"],
        "registration_time_start":"11:00",
        "registration_time_end":"16:00",
        "accepting_registrations":"on"
    },
    {
        "id":15,
        "preschool_name":"Little Explorers",
        "preschool_address":"123 Main Street, Philadelphia, PA 19101",
        "registration_days":["Mon","Wed","Fri"],
        "registration_time_start":"11:00",
        "registration_time_end":"16:00",
        "accepting_registrations":"on"
    }
]
```

### Request 4: Get Preschools based on registration date and time but no available preschools

- URL: `http://preschool-registration.local/wp-json/preschool-reg/v1/filter?registration_time=2023-06-03T13:00:00`
- Method: GET

### Expected Response

```json
[]
```

The time would have to be passed as 24 hour.

# Design Considerations and Choices


1. Utilizing the ACF Plugin for Custom Post Types and Fields:
When creating a custom post type with custom fields and REST API integration, I could choose to use the Advanced Custom Fields (ACF) plugin. ACF provides a user-friendly interface that allows to define custom post types and their associated fields easily. By utilizing ACF, I could configure the desired fields and enable REST API support for my custom post type conveniently. However, it's important to note that this approach may limit control over the underlying code and functionality, as well as access to advanced customization options and certain field types available only in the premium version of the plugin.

2. Custom Code Approach for Custom Post Types and ACF for Fields:
For a more hands-on approach, I could opt to write the code myself to create the custom post type and use the ACF plugin specifically for managing the custom fields. This method grants complete control over the code and functionality, allowing for extensive customization possibilities. I could define the custom post type using code, register the necessary endpoints for the REST API, and seamlessly utilize ACF to manage the custom fields associated with the custom post type. This approach provides flexibility in terms of field types and extends the capabilities beyond what is offered by default or in the free version of ACF.

While the ACF plugin offers a convenient and user-friendly solution, the custom code approach empowers me with granular control and flexibility over the functionality and field types, enabling me to tailor the solution to my exact needs.

However, I decided to do it all by writing code and not relying on another plugin. This gives me complete control over the code and functionality.

**The code for creating the custom post type is pretty straight forward.**

- A function to register the custom post type. 
- An array of labels is created to define the names and descriptions for various aspects of the custom post type, such as its singular and plural names, menu name, and more. These labels provide user-friendly text for different parts of the WordPress admin interface related to the custom post type.
- Another array called `$args` is created to pass the required arguments for registering the custom post type.

**Moving on to creating custom fields.**

- The function `add_preschool_registration_custom_fields()` is defined to add a meta box to the custom post type edit screen. This meta box will contain the custom fields for the "Preschool Registration" custom post type. 
- The `render_preschool_registration_fields()` function is defined as the callback function for rendering the custom fields HTML inside the meta box. It retrieves the current values of the custom fields using `get_post_meta()` and assigns them to variables for later use. 
- Inside the function, the HTML markup for each custom field is output using PHP. The values of the custom fields are pre-filled using the retrieved values, ensuring they are displayed correctly when editing the custom post. 
- The function `save_preschool_registration_custom_fields()` is defined to handle the saving of the custom field values. It checks if each custom field value is set in the `$_POST data`, sanitizes the values using appropriate sanitization functions, and uses `update_post_meta()` or `delete_post_meta()` to save or remove the custom field values for the current post.

**Next is extending the CPT REST API to return the fields based on Post ID**

- The function `extend_preschool_registration_rest_api()` is defined to register a new REST API route for retrieving the custom fields of a "Preschool Registration" post.
- Inside the register_rest_route() function, the following parameters are provided:
    - 'preschool-reg/v1' is the namespace for the REST API route.
    - '/(?P<id>\d+)' is the route pattern that expects a numeric ID as a URL parameter.
    - An array is passed as the third parameter, defining the HTTP method ('GET') and the callback function ('get_preschool_registration') that will handle the request.
- The `get_preschool_registration()` function is defined as the callback function for retrieving the custom fields of a "Preschool Registration" post based on the provided ID. The function then retrieves the post object using `get_post()` based on the provided ID.
- An array called $response is created, and the custom field values are retrieved using `get_post_meta()` and assigned to their respective keys in the array.

**Lastly, extending the CPT REST API to accept a datetime query parameter “registration_time” that filters all available preschools to only those that are ready for registration and are available at the requested day / time of the week**

- The function `extend_preschool_registration_rest_api()` is defined to register a new REST API route for filtering the "Preschool Registration" posts based on a datetime parameter.
- Inside the register_rest_route() function, the following parameters are provided:
  - 'preschool-reg/v1' is the namespace for the REST API route.
  - '/filter' is the route pattern for the filter. 
  - An array is passed as the third parameter, defining the HTTP method ('GET') and the callback function ('get_filtered_preschool_registrations') that will handle the request.
- The `get_filtered_preschool_registrations()` function is defined as the callback function for filtering the "Preschool Registration" posts based on the provided datetime parameter. 
- The function then creates an array called `$args` which contains the conditions to be met for the filtering.
- While there are posts in the query, the function iterates through each post and creates an array called `$response` containing the required custom field values. Each `$response` array is added to the `$responses` array.

This is the way this plugin is designed and implemented. 

Please feel free to reach out for any help! Thanks!
