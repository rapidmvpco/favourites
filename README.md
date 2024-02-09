Here is the code for the custon actions and Postgres queries to make make this work in your projects:
Remember to change any parameter names, parameter types and function names to the ones you are using.

#Flutterflow 

This is the custom action for retreiving the favourite items list on page load

```
Future<List<dynamic>> getFavourites(String user) async {
  try {
    final supabase = SupaFlow.client;
    final response = await supabase.rpc('get_favourites', params: {
      'p_user_id': user,
    });
    // Print the raw response for debugging purposes
    print('Raw Response: $response');

    if (response is List &&
        response.isNotEmpty &&
        response[0].containsKey('favourites')) {
      FFAppState().update(() {
        FFAppState().favourites = List<int>.from(response[0]['favourites']);
      });
    } else {
      print('Invalid or missing "favourites" key in the response');
    }
    return response;
  } catch (e) {
    print('Exception: $e');
    // Return an empty list in case of an exception
    return [];
  }
}

```

This is the custom action for appending an item id to the Supabase array:

```
Future<void> addFavourite(
  int jobid,
  String userid,
) async {
  try {
    final supabase = SupaFlow.client;

    await supabase.rpc('appendfav', params: {
      'p_jobid': jobid,
      'p_userid': userid,
    });

    //print('RPC function appendTeam called successfully');
  } catch (e) {
    print('Exception: $e');
    // Handle any exceptions if needed
  }
}
```

This is the custon action for removing an item from the Supabase array:

```
Future<void> removeFavourite(
  int jobid,
  String userid,
) async {
  try {
    final supabase = SupaFlow.client;

    await supabase.rpc('removefav', params: {
      'p_jobid': jobid,
      'p_userid': userid,
    });

    //print('RPC function appendTeam called successfully');
  } catch (e) {
    print('Exception: $e');
    // Handle any exceptions if needed
  }
}
```

#Supabase

This is the function for returning the list of favourites as a JSON on page load:

```
CREATE OR REPLACE FUNCTION get_favourites(p_user_id uuid)
RETURNS JSON
AS $$
DECLARE
    result_json JSON;
BEGIN
    SELECT COALESCE(json_agg(row_to_json(t)), '[]')::json
    INTO result_json
    FROM (
        SELECT favourites
        FROM users
        WHERE userid = p_user_id
    ) t;

    RETURN result_json;
END;
$$ LANGUAGE plpgsql;
```

This is the function for appending a favourited item to the array:

```
CREATE OR REPLACE FUNCTION public.appendFav(
  "p_jobid" int,
  "p_userid" uuid
) RETURNS VOID AS $$
BEGIN
  UPDATE public.users
  SET favourites = array_append(favourites, p_jobid)
  WHERE userid = p_userid;
END;
$$ LANGUAGE plpgsql;
```

This is the function for removing a favourited item from the array:

```
CREATE OR REPLACE FUNCTION public.removefav(
  "p_jobid" int,
  "p_userid" uuid
) RETURNS VOID AS $$
BEGIN
  UPDATE public.users
  SET favourites = array_remove(favourites, p_jobid)
  WHERE userid = p_userid;
END;
$$ LANGUAGE plpgsql;
```
