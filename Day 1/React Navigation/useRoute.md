Every screen in a navigator has a route object. It describes this screen’s current state: its name, its unique key, and any params passed to it, useRoute is a hook that returns that route object from any component

What’s inside the route object:

    {
        key: 'Details-a8f3k2',        // unique ID of this screen instance
        name: 'Details',              // the name you registered
        params: { id: 42, title: 'Rose' },  // data passed when navigating (may be undefined)
        path: undefined               // only set when opened via a deep link
    }


function HomeScreen({ navigation }) {
  return (
    <Button
      title="Open Rose"
      onPress={() => navigation.navigate('Details', { id: 42, title: 'Rose' })}
    />
  );
}

function DetailsScreen({ route }) {
  const { id, title } = route.params;
  return <Text>{title} (#{id})</Text>;
}


route prop vs useRoute
	route prop	useRoute()
Available in	Screen components only	Any component inside a screen
Prop drilling	Needed for children	Not needed
Best for	Simple screens	Shared UI, headers, deep components