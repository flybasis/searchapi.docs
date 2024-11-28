<script>
  document.addEventListener("DOMContentLoaded", () => {
    const link = document.querySelector("a");
    // remove the link from the DOM
    link.remove();
  });
</script>

<div style="display: flex; align-items: center; justify-content: space-between; margin: 20px 0;">
  <img src="images/black-large.webp" alt="Flybasis Logo" style="width: 100px; margin-right: 20px;">

  <div style="display: flex; gap: 10px;">
    <a href="https://flybasis.com">Flybasis</a>
    <a href="/searchapi.docs">WebSocket API</a>
    <a href="/searchapi.docs/checkout-api">Checkout API</a>
  </div>
</div>

# Flybasis WebSocket API Documentation

Welcome to the Flybasis WebSocket API documentation. This API provides real-time communication capabilities for flight search and booking services using WebSocket technology. It is designed to offer seamless and efficient data exchange, ensuring a responsive and interactive user experience. Below, you'll find detailed information on how to connect, authenticate, and interact with the various endpoints available in our API.

## Table of Contents

- [Flybasis WebSocket API Documentation](#flybasis-websocket-api-documentation)
  - [Table of Contents](#table-of-contents)
  - [Host and Route](#host-and-route)
  - [Events](#events)
  - [Authentication](#authentication)
    - [Authentication Process](#authentication-process)
  - [Input Parameters](#input-parameters)
    - [Examples](#examples)
      - [1- One-way search](#1--one-way-search)
      - [2- Round-trip search](#2--round-trip-search)
  - [Output type](#output-type)
    - [`flights` event](#flights-event)
    - [`prices` event (experimental/buggy)](#prices-event-experimentalbuggy)

## Host and Route

`https://api.flybasis.com/sockets/v1/stream-flights`

## Events

| Event Name | Emitted By | In-response to | Description                                                               |
| ---------- | :--------: | :------------: | ------------------------------------------------------------------------- |
| `search`   |   Client   |       -        | When a new search is requested using the [Parameters](#input-parameters). |
| `flight`   |   Server   |    `search`    | When a new flight is found that matches the search parameters.            |
| `error`    |   Server   |       -        | When an error occurs on the server when fetching data.                    |

## Authentication

To ensure secure access to the Flybasis WebSocket API, all clients must authenticate before establishing a connection. Authentication is handled via a middleware that verifies the client's credentials.

### Authentication Process

1. **Obtain an API Key**: Users must first obtain an API key from the Flybasis developer portal. This key is unique to each user and is required for all API requests.

2. **Connect to the WebSocket Server**: When connecting to the WebSocket server, include the API key in the connection request. This can typically be done by adding the key as a query parameter or in the headers, depending on the server configuration.

3. **Middleware Verification**: Upon connection, the authentication middleware will intercept the request and verify the provided API key. If the key is valid, the connection is established, and the user can begin interacting with the API.

4. **Error Handling**: If the API key is invalid or missing, the connection will be rejected, and an error message will be returned to the client.

## Input Parameters

The API can be operated in two modes:

1. One-way search
2. Round-trip search

The explanation and valid values for each parameter are provided below.

| Parameter Name | Type   | Description                                                                                                                                                                                                                                            | Valid Values                                                                                                                       |
| -------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `tripType`     | string | The type of trip the user wants to search for. Can be either "oneway" or "roundtrip".                                                                                                                                                                  | `oneway`, `roundtrip`                                                                                                              |
| `cabin`        | string | The cabin class the user wants to search for.                                                                                                                                                                                                          | `Economy`, `Premium Economy`, `Business`, `First`, `Main Economy`                                                                  |
| `programs`     | array  | The airline programs the user wants to search for.                                                                                                                                                                                                     | `AM`, `AC`, `KL`, `AS`, `AA`, `AV`, `BA`, `CM`, `DL`, `EK`, `EY`, `IB`, `B6`, `QF`, `SK`, `SQ`, `NK`, `TP`, `TK`, `UA`, `VS`, `VA` |
| `stops`        | array  | The stops the user wants to search for. For "oneway" this will be just a single valued array. For "roundtrip" this would contain two stop objects. The origin and destination fields would be interchanged for the second stop object for "roundtrip". | `StopObject`, details below                                                                                                        |

**Stop Object**

| Parameter Name | Type   | Description                                        | Valid Values                                   |
| -------------- | ------ | -------------------------------------------------- | ---------------------------------------------- |
| `origin`       | array  | The origin city the user wants to search for.      | A list of (IATA codes for) origin cities.      |
| `destination`  | array  | The destination city the user wants to search for. | A list of (IATA codes for) destination cities. |
| `date`         | object | The date the user wants to search for.             | `DateObject`, details below                    |

**Date Object**

| Parameter Name | Type   | Description                                      | Valid Values                                                                                                                                                                                              |
| -------------- | ------ | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `value`        | string | The date the user wants to search for.           | A date string in the format `YYYY-MM-DD`.                                                                                                                                                                 |
| `range`        | number | The range of dates the user wants to search for. | A number representing the number of days before and after the `value` date. For example, for value =`2024-04-12` and range = 3, the search will be for flights departing on `2024-04-09` to `2024-04-15`. |

### Examples

#### 1- One-way search

The user wants to search for only one-way flights.

```typescript
// One way search

{
  "tripType": "oneway",
  "cabin": "Business",
  "programs": ["AC", "KL", "AS", "AA"],
  "stops": [
    {
      "origin": ["JFK", "LAX"],
      "destination": ["LON", "PAR"],
      "date": {
        "value": "2024-12-15",
        "range": 0
      }
    }
  ]
}
```

#### 2- Round-trip search

The user wants to search for round-trip flights.

```typescript
// Two way search

{
  "tripType": "roundtrip",
  "cabin": "Business",
  "programs": ["AC", "KL", "AS", "AA"],
  "stops": [
    {
      "origin": ["JFK", "LAX"],
      "destination": ["LON", "PAR"],
      "date": {
        "value": "2024-12-15",
        "range": 0
      }
    },
    {
      "origin": ["LON", "PAR"],
      "destination": ["JFK", "LAX"],
      "date": {
        "value": "2024-12-15",
        "range": 0
      }
    }
  ]
}
```

## Output type

### `flights` event

From the `flights` event, the following data is returned.

```typescript
{
  "flights": Flight[]
}
```

**`Flight Object`**

```typescript
type Flight = {
  id: string;
  date: string; // YYYY-MM-DD
  odKey: string;
  retail: number; // Retail price of the flight in USD
  surcharge: number;
  distance: number;
  cabin_type: string;
  airline_name: string;
  program_code: string;
  airline_code: string;
  percent_premium: number;
  award_points: number;
  travel_minutes_total: number;
  products: FlightProduct[]; // Use the `FlightProduct` type defined earlier
  stops: number;
  search_origin: string;
  search_destination: string;
  origin: string;
  origin_city: string;
  destination: string;
  destination_city: string;
  program_name: string;
  cost: number;
  risks: {
    long_layover: boolean;
    risky_connection: boolean;
    overnight: boolean;
  };
  url: string | null;
  program_list:
    | {
        program_code: string;
        award_points: number;
        cost: number;
        surcharge: number;
        url: string;
      }[]
    | null;
};

type FlightProduct = {
  layover_time: number;
  arrival_time: string;
  distance: number | null;
  airline_code: string;
  flight_number: string;
  origin: string;
  aircraft: string;
  destination: string;
  travel_minutes: number;
  cabin_type: "Economy" | "Premium Economy" | "Business" | "First";
  departure_time: string;
  origin_city: string;
  destination_city: string;
  origin_airport: string;
  destination_airport: string;
  airline_name: string;
};
```

### `prices` event (experimental/buggy)

The `prices` event is emitted when the retail price of a flight is updated.
