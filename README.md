# kptncook-api-reverse-engineering
I got this HTTP-API documentation by reversing the kptncook app and analysing its traffic.  
Check out https://medium.com/analytics-vidhya/reversing-and-analyzing-the-cooking-app-kptncook-my-recipe-collection-5b5b04e5a085 for a guide on how to get started with the kpntcook app.

The API is accessible under https://mobile.kptncook.com

## Optaining API key
Each call to the api (aswell as calls to get objects such as recipe images from cloudfront) have to include a http param called kptnkey. This kptnkey is hardcoded in the app and doesn't seem to change.

I'm not sure what the implications of providing such an API key are so I won't provide it here. You can easily get it by analyzing the app in android-studio or just by searching `kptnkey` on github.

Then just append this key to each call like this: `https://mobile.kptncook.com/<ressource_id>/?kptnkey=6*****y-o**k-I*****J-j****6`


## Login
Logs user in and returns user info, the user's access token, name and the user's favorite recipe's ids.
* **kptnkey required:**  
    No
* **URL:**  
    `/login/userpass`
* **Method:**  
    `POST`
* **Request Body:**  
    **type**: json  
    **example**:  
    ```json
    {
        "email": "<account_address>",
        "password": "<cleartext_password>"
    }
    ```
* **Request Answer:**  
    **example**:  
    ```json
    {
        "accessToken": "1*****5-8**3-4**1-5**b-e*********d",
        "name": "<realname>",
        "favspace": 35,
        "favorites": [
            "5dcbd5cc5400009587b05515",
            ...
        ],
        "inviteCode": "4****4"
    }
    ```

## Favorites
Gets all favorites of the logged in user and returns their IDs as a list
* **kptnkey required:**  
    No
* **URL:**  
    `/favorites`
* **Method:**  
    `GET`
* **Request Headers:**  
   `Token`: insert the accessToken returned when logging in here  
   **example**: `Token: 1*****5-8**3-4**1-5**b-e*********d`
* **Request Answer:**  
    **example**:  
    ```json
    {
        "favspace": 35,
        "favorites": [
            "52c5e4021d46335f01572524",
            ...
        ]
    }
    ```

## Search
Returns one recipe by its id. You can search by a recipe's uid or identifier:  
* **Possible ids:**
    * `uid`: Is the id which can be extracted from share links  
        e.g. `http://mobile.kptncook.com/recipe/pinterest/Low-Carb-Tarte-Flamb%C3%A9e-with-Serrano-Ham-%26-Cream-Cheese/315c3c32)`  
        The `315c3c32` part at the end is the uid
    * `identifier`: Is the _internal id_  which is returned by the other api functions in the `$oid` field
* **kptnkey required:**  
    Yes
* **URL:**  
    `/recipes/search`
* **Method:**  
    `POST`
* **Request Headers:**  
   `Token`: insert the accessToken returned when logging in here  
   **example**: `Token: 1*****5-8**3-4**1-5**b-e*********d`
* **Request Parameters:**  
   `kptnkey`  
   **example**: `?kptnkey=6*****y-o**k-I*****J-j****6`
* **Request Body:**  
    **type**: json  
    **example**:  
    ```json
    [
        {
            "uid": "315c3c32"
        }
    ]
    ```
    or 
    ```json
    [
        {
            "identifier": "59b68c8d2c00000b0535125a"
        }
    ]
    ```
* **Full Request Example:**
    ```bash
    curl --location --request POST 'https://mobile.kptncook.com/recipes/search?kptnkey=6*****y-o**k-I*****J-j****6' \
    --header 'Token: 1*****5-8**3-4**1-5**b-e*********d' \
    --header 'Content-Type: application/json' \
    --data-raw '[
            {
                "uid": "315c3c32"
            }
            ]
    '
    ```
* **Request Answer:**  
    **example**:  
    ```json
    [
        {
            "_id": {
                "$oid": "59b68c8d2c00000b0535125a"
            },
            "localizedTitle": {
                ...
                "en": "Low Carb Tarte Flambée with Serrano Ham & Cream Cheese",
                ...
            },
            ...
        },
        ...
    ]
    ```

## Daily Favorites 
Returns the 3 daily featured recipes in full detail
* **kptnkey required:**  
    Yes
* **URL:**  
    `/reciped/de/<unix_timestamp>`  
    The timestamp doesn't actually matter, it always returns the 3 current recipes  
    (a value of 1 can also be used)
* **Method:**  
    `GET`
* **Request Parameters:**  
   `kptnkey`  
   **example**: `?kptnkey=6*****y-o**k-I*****J-j****6`
* **Request Answer:**  
    **example**:  
    ```json
    [
        {
            "_id": {
                "$oid": "60b0c1323c000035008250c9"
            },
            "localizedTitle": {
                ...
                "en": "Cheesy Chicken Bake with Garlic Croutons",
                ...
            },
            ...
        },
        ...
    ]
    ```