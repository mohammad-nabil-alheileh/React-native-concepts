The components That build the React Native apps

1.View: 
    The basic container, like (div). used for layout and grouping.
        <View>
            ...
        </View>

2.Text: 
    All text must be inside <Text>. Writing a string directly inside a <View> throws the error ("Text strings must be rendered within a <Text> component"), a red error screen in development, and it can crash the app in production.
        <Text>Mohammad</Text>

    Nested Text inherits styles from its parent Text only. A <View> does NOT pass styles (like color or fontSize) down to the Text inside it.
    It supports (numberOfLines) and (ellipsizeMode (head, middle, tail, clip)) for truncation.

3.Image:
    to display images in my app (require(...) is what imports the file)
        <Image source={require('./assets/logo.png')}/>
    
    Local images have their size detected automatically, and resizeMode (cover, contain, stretch, center, repeat) is easy to use, But Remote images need explicit dimensions.
        <Image source={{ uri: 'https://example.com/photo.png' }} style={{ width: 100, height: 100 }}/>

4.ScrollView:
    Renders all its children at once and makes them scrollable.
        <ScrollView contentContainerStyle={{ padding: 16 }}>
            <Text>Item 1</Text>
            <Text>Item 2</Text>
            {/* ...more content */}
        </ScrollView>

    Everything is rendered up front, so it is bad for long lists because memory and performance suffer. For long lists use (FlatList), which only renders the items visible on screen.

5.TextInput:
    catch the input of the user
        <TextInput
          value={name}
          onChangeText={setName}
          placeholder="Your name"
          keyboardType="default"
          secureTextEntry={false}
        />

    (onChangeText) gives you the string directly (no event.target.value). It has built-in props such as (keyboardType, secureTextEntry, and autoCapitalize). Behavior differs slightly between iOS and Android. The keyboard can cover the input, which you handle with (KeyboardAvoidingView)

6.Pressable:
    The way to handle touches
        <Pressable
            onPress={() => console.log('pressed')}
            onLongPress={() => console.log('long press')}
            style={({ pressed }) => [
                { padding: 12 },
                pressed && { opacity: 0.5 },
            ]}
        >
            <Text>Press me</Text>
        </Pressable>

    It is flexible, since style receives a pressed state. It supports (press, long press, and hover (onHoverIn, onHoverOut — mainly web/desktop)) events, and it works on any child. It has no default styling or feedback, so you build everything yourself, except on Android where you can add a native ripple with (android_ripple={{ color: '#ccc' }}).

7.Modal:
    It's a component that displays content on top of the current screen.
        <Modal visible={visible} animationType="slide" transparent onRequestClose={() => setVisible(false)}>
            <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', backgroundColor: 'rgba(0,0,0,0.5)' }}>
                <View style={{ backgroundColor: 'white', padding: 20, borderRadius: 8 }}>
                    <Text>I'm a modal</Text>
                    <Pressable onPress={() => setVisible(false)}><Text>Close</Text></Pressable>
                </View>
            </View>
        </Modal>

    It is native, with built-in animations (slide, fade, none), and onRequestClose handles the Android back button.
    With (transparent), the Modal has no background, so you style the dark overlay yourself (the outer View above).
