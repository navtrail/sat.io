import React, {
    useState,
    useEffect,
    createContext,
    useContext
} from 'react';
import {
    NavigationContainer
} from '@react-navigation/native';
import AppNavigator from './navigation/AppNavigator';
import {
    ThemeProvider,
    useTheme
} from './styles/theme';
import LoadingIndicator from './components/LoadingIndicator';
import ErrorMessage from './components/ErrorMessage';

// Create ThemeContext
const ThemeContext = createContext({
    themeMode: 'light',
    toggleTheme: () => {}
});

// Custom hook to access the theme
export const useAppTheme = () => useContext(ThemeContext);

const App = () => {
    const [isLoading, setIsLoading] = useState(true);
    const [error, setError] = useState(null);
    const [themeMode, setThemeMode] = useState('light'); // Default theme

    useEffect(() => {
        // Simulate initial loading (fetching data, etc.)
        const loadData = async () => {
            try {
                // You can simulate an API call or any loading process here
                await new Promise(resolve => setTimeout(resolve, 2000)); // Simulate 2 seconds loading
            } catch (err) {
                setError(err);
            } finally {
                setIsLoading(false);
            }
        };

        loadData();
    }, []);

    const toggleTheme = () => {
        setThemeMode(themeMode === 'light' ? 'dark' : 'light');
    };

    if (isLoading) {
        return < LoadingIndicator / > ;
    }

    if (error) {
        return < ErrorMessage message = {
            error.message
        }
        onRetry = {
            () => setIsLoading(true)
        }
        />;
    }

    return ( <
        ThemeProvider themeMode = {
            themeMode
        }
        toggleTheme = {
            toggleTheme
        } >
        <
        NavigationContainer >
        <
        AppNavigator / >
        <
        /NavigationContainer> <
        /ThemeProvider>
    );
};

export default App;


import React from 'react';
import {
    createStackNavigator
} from '@react-navigation/stack';
import MapScreen from '../screens/MapScreen';
import PlaceDetailsScreen from '../screens/PlaceDetailsScreen';
import ProfileScreen from '../screens/ProfileScreen';
import {
    useTheme
} from '../styles/theme'; // Import the useTheme hook

const Stack = createStackNavigator();

const AppNavigator = () => {
    const {
        theme
    } = useTheme(); // Access the theme
    return ( <
        Stack.Navigator initialRouteName = "Map"
        screenOptions = {
            {
                headerStyle: {
                    backgroundColor: theme.primary,
                },
                headerTintColor: theme.text,
                headerTitleStyle: {
                    fontWeight: 'bold',
                },
            }
        } >
        <
        Stack.Screen name = "Map"
        component = {
            MapScreen
        }
        options = {
            {
                title: 'Map'
            }
        }
        /> <
        Stack.Screen name = "PlaceDetails"
        component = {
            PlaceDetailsScreen
        }
        options = {
            {
                title: 'Place Details'
            }
        }
        /> <
        Stack.Screen name = "Profile"
        component = {
            ProfileScreen
        }
        options = {
            {
                title: 'Profile'
            }
        }
        /> <
        /Stack.Navigator>
    );
};

export default AppNavigator;



import React, {
    createContext,
    useContext,
    useState
} from 'react';
import {
    StyleSheet
} from 'react-native';

// Create theme context
const ThemeContext = createContext({
    themeMode: 'light',
    toggleTheme: () => {}
});

// Custom hook to access the theme context
export const useTheme = () => useContext(ThemeContext);

// Theme provider component
export const ThemeProvider = ({
    children,
    themeMode,
    toggleTheme
}) => {
    const theme = themeMode === 'light' ? lightTheme : darkTheme;

    return ( <
        ThemeContext.Provider value = {
            {
                themeMode,
                theme,
                toggleTheme
            }
        } > {
            children
        } <
        /ThemeContext.Provider>
    );
};

// Light theme
export const lightTheme = StyleSheet.create({
    background: '#f0f0f0',
    text: '#000000',
    primary: '#6200ee',
    secondary: '#03dac6',
    accent: '#8bc34a',
    error: '#b00020',
    cardBackground: '#ffffff', // Added card background
    inputBorder: '#cccccc', // Added input border color
});

// Dark theme
export const darkTheme = StyleSheet.create({
    background: '#303030',
    text: '#ffffff',
    primary: '#bb86fc',
    secondary: '#03dac6',
    accent: '#8bc34a',
    error: '#cf6679',
    cardBackground: '#424242', // Dark card background
    inputBorder: '#666666', // Dark input border color
});


import React from 'react';
import {
    View,
    ActivityIndicator,
    StyleSheet
} from 'react-native';
import {
    useTheme
} from '../styles/theme'; // Import the useTheme hook

const LoadingIndicator = () => {
    const {
        theme
    } = useTheme(); // Access the theme

    return ( <
        View style = {
            [styles.container, {
                backgroundColor: theme.background
            }]
        } >
        <
        ActivityIndicator size = "large"
        color = {
            theme.primary
        }
        /> <
        /View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
});

export default LoadingIndicator;




import React from 'react';
import {
    View,
    Text,
    Button,
    StyleSheet
} from 'react-native';
import {
    useTheme
} from '../styles/theme'; // Import the useTheme hook

const ErrorMessage = ({
    message,
    onRetry
}) => {
    const {
        theme
    } = useTheme(); // Access the theme

    return ( <
        View style = {
            [styles.container, {
                backgroundColor: theme.background
            }]
        } >
        <
        Text style = {
            [styles.text, {
                color: theme.error
            }]
        } > Error: {
            message
        } < /Text> {
            onRetry && < Button title = "Retry"
            onPress = {
                onRetry
            }
            color = {
                theme.primary
            }
            />} <
            /View>
        );
    };

    const styles = StyleSheet.create({
        container: {
            flex: 1,
            justifyContent: 'center',
            alignItems: 'center',
            padding: 20,
        },
        text: {
            fontSize: 16,
            marginBottom: 10,
            textAlign: 'center',
        },
    });

    export default ErrorMessage;





   import React, {
        useState,
        useEffect,
        useRef
    } from 'react';
    import MapView, {
        Marker,
        Callout
    } from 'react-native-maps';
    import {
        StyleSheet,
        View,
        Alert,
        Text
    } from 'react-native';
    import * as Location from 'expo-location';
    import {
        useTheme
    } from '../styles/theme'; // Import the useTheme hook
    import {
        fetchNearbyPlaces
    } from '../utils/api';

    const MapViewComponent = ({
            onMarkerPress
        }) => {
            const {
                theme
            } = useTheme(); // Access the theme
            const [userLocation, setUserLocation] = useState(null);
            const [errorMsg, setErrorMsg] = useState(null);
            const [nearbyPlaces, setNearbyPlaces] = useState([]);
            const mapRef = useRef(null); // Ref to the MapView

            useEffect(() => {
                (async () => {
                    let {
                        status
                    } = await Location.requestForegroundPermissionsAsync();
                    if (status !== 'granted') {
                        setErrorMsg('Permission to access location was denied');
                        return;
                    }

                    try {
                        let location = await Location.getCurrentPositionAsync({});
                        setUserLocation(location.coords);
                        // Fetch nearby places when the user's location is available
                        fetchNearby(location.coords);
                    } catch (err) {
                        setErrorMsg('Unable to get current location.');
                    }
                })();
            }, []);

            useEffect(() => {
                if (errorMsg) {
                    Alert.alert('Location Error', errorMsg);
                }
            }, [errorMsg]);

            const fetchNearby = async (coords) => {
                try {
                    const places = await fetchNearbyPlaces(coords.latitude, coords.longitude);
                    setNearbyPlaces(places);
                } catch (error) {
                    console.error("Error fetching nearby places:", error);
                    Alert.alert('Error', 'Failed to fetch nearby places.');
                }
            };

            // Use useEffect to trigger map zoom after the component is mounted and userLocation is available
            useEffect(() => {
                if (userLocation && mapRef.current) {
                    mapRef.current.animateToRegion({
                        latitude: userLocation.latitude,
                        longitude: userLocation.longitude,
                        latitudeDelta: 0.02, // You can adjust these values to fit more locations
                        longitudeDelta: 0.02,
                    }, 2000); // The zoom animation will take 2 seconds
                }
            }, [userLocation]);

            let initialRegion = null;
            if (userLocation) {
                initialRegion = {
                    latitude: userLocation.latitude,
                    longitude: userLocation.longitude,
                    latitudeDelta: 0.0922,
                    longitudeDelta: 0.0421,
                };
            } else {
                initialRegion = {
                    latitude: 37.78825,
                    longitude: -122.4324,
                    latitudeDelta: 0.0922,
                    longitudeDelta: 0.0421,
                };
            }

            return ( <
                View style = {
                    styles.container
                } >
                <
                MapView ref = {
                    mapRef
                }
                style = {
                    styles.map
                }
                initialRegion = {
                    initialRegion
                }
                showsUserLocation = {
                    true
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Map of nearby locations"
                accessibilityHint = "Displays markers for nearby places and user's current location" > {
                    nearbyPlaces.map((location) => ( <
                        Marker key = {
                            location.place_id
                        }
                        coordinate = {
                            {
                                latitude: location.geometry.location.lat,
                                longitude: location.geometry.location.lng
                            }
                        }
                        title = {
                            location.name
                        }
                        description = {
                            location.vicinity
                        }
                        pinColor = {
                            theme.primary
                        } // Use theme color for pin
                        onPress = {
                            () => onMarkerPress(location)
                        }
                        accessible = {
                            true
                        }
                        accessibilityLabel = {
                            `Marker for ${location.name}`
                        }
                        accessibilityHint = {
                            `Double tap to view details of ${location.name}`
                        } >
                        <
                        Callout tooltip accessible = {
                            true
                        }
                        accessibilityLabel = {
                            `Callout for ${location.name}`
                        }
                        accessibilityHint = {
                            `Displays additional information about ${location.name}. Double tap to view details.`
                        } >
                        <
                        View style = {
                            styles.calloutView
                        } >
                        <
                        Text style = {
                            styles.calloutText
                        } > {
                            location.name
                        } < /Text> <
                        /View> <
                        /Callout> <
                        /Marker>
                    ))
                } <
                /MapView> <
                /View>
            );
        };

        const styles = StyleSheet.create({
            container: {
                flex: 1,
            },
            map: {
                width: '100%',
                height: '100%',
            },
            calloutView: {
                backgroundColor: 'rgba(255, 255, 255, 0.8)',
                borderRadius: 10,
                padding: 5
            },
            calloutText: {
                fontSize: 16,
                fontWeight: 'bold'
            }
        });

        export default MapViewComponent;





       import React, {
            useState,
            useEffect,
            useRef
        } from 'react';
        import {
            View,
            TextInput,
            StyleSheet,
            FlatList,
            TouchableOpacity,
            Text,
            Keyboard
        } from 'react-native';
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook
        import {
            fetchPlaceSuggestions
        } from '../utils/api';

        const SearchBar = ({
            onSearch,
            onSuggestionPress
        }) => {
            const {
                theme
            } = useTheme(); // Access the theme
            const [searchText, setSearchText] = useState('');
            const [suggestions, setSuggestions] = useState([]);
            const inputRef = useRef(null); // Ref for the TextInput

            useEffect(() => {
                // Simulate fetching suggestions from an API (Debounced)
                const delayDebounceFn = setTimeout(async () => {
                    if (searchText) {
                        try {
                            const suggestions = await fetchPlaceSuggestions(searchText);
                            setSuggestions(suggestions);
                        } catch (error) {
                            console.error("Error fetching suggestions:", error);
                            // Handle error appropriately
                        }
                    } else {
                        setSuggestions([]);
                    }
                }, 500);

                return () => clearTimeout(delayDebounceFn); // Cleanup the timeout
            }, [searchText]);

            const handleSearchTextChange = (text) => {
                setSearchText(text);
            };

            const clearInput = () => {
                setSearchText('');
                setSuggestions([]);
                Keyboard.dismiss();
                if (inputRef.current) {
                    inputRef.current.focus(); // Focus on the input after clearing
                }
            };

            return ( <
                View style = {
                    [styles.container, {
                        backgroundColor: theme.cardBackground
                    }]
                } >
                <
                TextInput ref = {
                    inputRef
                } // Attach the ref to the TextInput
                style = {
                    [styles.input, {
                        color: theme.text,
                        borderColor: theme.inputBorder
                    }]
                }
                placeholder = "Search for places"
                placeholderTextColor = {
                    theme.text
                }
                value = {
                    searchText
                }
                onChangeText = {
                    handleSearchTextChange
                }
                onSubmitEditing = {
                    () => {
                        onSearch(searchText);
                        Keyboard.dismiss();
                    }
                }
                clearButtonMode = "while-editing"
                accessibilityLabel = "Search"
                accessibilityHint = "Enter a place name to search for locations" /
                >
                <
                FlatList data = {
                    suggestions
                }
                keyExtractor = {
                    (item) => item.place_id
                }
                renderItem = {
                    ({
                        item
                    }) => ( <
                        TouchableOpacity style = {
                            [styles.suggestion, {
                                borderBottomColor: theme.inputBorder
                            }]
                        }
                        onPress = {
                            () => {
                                onSuggestionPress(item);
                                clearInput();
                            }
                        }
                        accessible = {
                            true
                        }
                        accessibilityLabel = {
                            `Suggestion: ${item.description}`
                        }
                        accessibilityHint = "Double tap to select this suggestion" >
                        <
                        Text style = {
                            {
                                color: theme.text
                            }
                        } > {
                            item.description
                        } < /Text> <
                        /TouchableOpacity>
                    )
                }
                keyboardShouldPersistTaps = "handled" /
                >
                <
                /View>
            );
        };

        const styles = StyleSheet.create({
            container: {
                padding: 10,
                backgroundColor: '#fff',
            },
            input: {
                height: 40,
                borderColor: 'gray',
                borderWidth: 1,
                paddingLeft: 10,
                borderRadius: 5,
                marginBottom: 5,
                fontSize: 16,
            },
            suggestion: {
                padding: 10,
                borderBottomWidth: 1,
                borderBottomColor: '#eee',
            },
        });

        export default SearchBar;


        import React from 'react';
        import {
            View,
            Text,
            Image,
            StyleSheet,
            TouchableOpacity
        } from 'react-native';
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook

        const PlaceCard = ({
            place,
            onPress
        }) => {
            const {
                theme
            } = useTheme(); // Access the theme

            return ( <
                TouchableOpacity style = {
                    [styles.card, {
                        backgroundColor: theme.cardBackground
                    }]
                }
                onPress = {
                    onPress
                }
                accessible = {
                    true
                }
                accessibilityLabel = {
                    `Place: ${place.name}`
                }
                accessibilityHint = "Double tap to view details" >
                <
                Image style = {
                    styles.image
                }
                source = {
                    {
                        uri: place.photos ? place.photos[0].photo_reference : 'https://via.placeholder.com/150'
                    }
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Place Image" /
                >
                <
                View style = {
                    styles.details
                } >
                <
                Text style = {
                    [styles.name, {
                        color: theme.text
                    }]
                } > {
                    place.name
                } < /Text> <
                Text style = {
                    [styles.address, {
                        color: theme.text
                    }]
                } > {
                    place.vicinity
                } < /Text> <
                Text style = {
                    [styles.rating, {
                        color: theme.text
                    }]
                } > Rating: {
                    place.rating
                } < /Text> <
                /View> <
                /TouchableOpacity>
            );
        };

        const styles = StyleSheet.create({
            card: {
                backgroundColor: '#fff',
                borderRadius: 8,
                overflow: 'hidden',
                margin: 10,
                elevation: 3, // Shadow for Android
                shadowColor: '#000', // Shadow properties for iOS
                shadowOffset: {
                    width: 0,
                    height: 2
                },
                shadowOpacity: 0.25,
                shadowRadius: 3.84,
            },
            image: {
                height: 150,
                width: '100%',
            },
            details: {
                padding: 10,
            },
            name: {
                fontSize: 18,
                fontWeight: 'bold',
            },
            address: {
                fontSize: 14,
                color: 'gray',
            },
            rating: {
                fontSize: 14,
            },
        });

        export default PlaceCard;




       import React from 'react';
        import {
            View,
            Text,
            Image,
            StyleSheet,
            TouchableOpacity
        } from 'react-native';
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook
        import {
            useNavigation
        } from '@react-navigation/native'; // Import useNavigation

        const ProfileHeader = ({
            user
        }) => {
            const {
                theme
            } = useTheme(); // Access the theme
            const navigation = useNavigation();

            return ( <
                View style = {
                    [styles.header, {
                        backgroundColor: theme.primary
                    }]
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Profile Header"
                accessibilityHint = "Contains profile picture and user info" >
                <
                TouchableOpacity onPress = {
                    () => navigation.navigate('Profile')
                }
                accessible = {
                    true
                }
                accessibilityLabel = "View Profile"
                accessibilityHint = "Navigates to the user profile screen" >
                <
                Image style = {
                    styles.avatar
                }
                source = {
                    {
                        uri: user.avatar || 'https://via.placeholder.com/100'
                    }
                }
                accessible = {
                    true
                }
                accessibilityLabel = "User Profile Picture" /
                >
                <
                /TouchableOpacity> <
                View style = {
                    styles.info
                } >
                <
                Text style = {
                    [styles.name, {
                        color: theme.text
                    }]
                } > {
                    user.name
                } < /Text> <
                Text style = {
                    [styles.bio, {
                        color: theme.text
                    }]
                } > {
                    user.bio || 'No bio provided'
                } < /Text> <
                /View> <
                /View>
            );
        };

        const styles = StyleSheet.create({
            header: {
                flexDirection: 'row',
                alignItems: 'center',
                backgroundColor: '#6200ee',
                padding: 20,
            },
            avatar: {
                width: 80,
                height: 80,
                borderRadius: 40,
                marginRight: 20,
            },
            info: {
                flex: 1,
            },
            name: {
                fontSize: 20,
                fontWeight: 'bold',
            },
            bio: {
                fontSize: 16,
            },
        });

        export default ProfileHeader;



       import React, {
            useState,
            useEffect
        } from 'react';
        import {
            View,
            StyleSheet,
            SafeAreaView,
            Switch,
            Text
        } from 'react-native';
        import MapViewComponent from '../components/MapView';
        import SearchBar from '../components/SearchBar';
        import {
            useNavigation
        } from '@react-navigation/native'; // Import useNavigation
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook
        import * as Location from 'expo-location';

        const MapScreen = () => {
            const {
                theme,
                toggleTheme
            } = useTheme(); // Access the theme and toggle function
            const navigation = useNavigation(); // Use navigation hook
            const [hasLocationPermission, setHasLocationPermission] = useState(false);

            useEffect(() => {
                const checkLocationPermission = async () => {
                    let {
                        status
                    } = await Location.getForegroundPermissionsAsync();
                    setHasLocationPermission(status === 'granted');
                };

                checkLocationPermission();
            }, []);

            const handleSearch = (searchText) => {
                // Simulate searching for places
                console.log('Searching for:', searchText);
            };

            const handleSuggestionPress = (suggestion) => {
                // Simulate navigating to the selected place
                console.log('Suggestion pressed:', suggestion);
            };

            const handleMarkerPress = (location) => {
                navigation.navigate('PlaceDetails', {
                    place: location
                });
            };

            return ( <
                SafeAreaView style = {
                    [styles.safeArea, {
                        backgroundColor: theme.background
                    }]
                } >
                <
                View style = {
                    [styles.container, {
                        backgroundColor: theme.background
                    }]
                } >
                <
                View style = {
                    styles.switchContainer
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Theme Settings"
                accessibilityHint = "Allows switching between light and dark theme" >
                <
                Text style = {
                    {
                        color: theme.text
                    }
                } > Dark Mode: < /Text> <
                Switch value = {
                    theme.themeMode === 'dark'
                }
                onValueChange = {
                    toggleTheme
                }
                accessibilityLabel = "Dark Mode Switch"
                accessibilityHint = "Toggles between dark and light theme" /
                >
                <
                /View> <
                SearchBar onSearch = {
                    handleSearch
                }
                onSuggestionPress = {
                    handleSuggestionPress
                }
                /> <
                MapViewComponent onMarkerPress = {
                    handleMarkerPress
                }
                /> <
                /View> <
                /SafeAreaView>
            );
        };

        const styles = StyleSheet.create({
            safeArea: {
                flex: 1,
            },
            container: {
                flex: 1,
                backgroundColor: '#fff',
            },
            switchContainer: {
                flexDirection: 'row',
                justifyContent: 'space-between',
                alignItems: 'center',
                paddingHorizontal: 20,
                paddingTop: 10,
            },
        });

        export default MapScreen;





       import React from 'react';
        import {
            View,
            Text,
            Image,
            StyleSheet,
            ScrollView
        } from 'react-native';
        import {
            useRoute
        } from '@react-navigation/native';
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook

        const PlaceDetailsScreen = () => {
            const {
                theme
            } = useTheme(); // Access the theme
            const route = useRoute();
            const {
                place
            } = route.params;

            return ( <
                ScrollView style = {
                    [styles.container, {
                        backgroundColor: theme.background
                    }]
                } >
                <
                Image style = {
                    styles.image
                }
                source = {
                    {
                        uri: place.photos ? place.photos[0].photo_reference : 'https://via.placeholder.com/300'
                    }
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Place Image" /
                >
                <
                View style = {
                    [styles.details, {
                        backgroundColor: theme.cardBackground
                    }]
                } >
                <
                Text style = {
                    [styles.name, {
                        color: theme.text
                    }]
                } > {
                    place.name
                } < /Text> <
                Text style = {
                    [styles.address, {
                        color: theme.text
                    }]
                } > {
                    place.vicinity
                } < /Text> <
                Text style = {
                    [styles.rating, {
                        color: theme.text
                    }]
                } > Rating: {
                    place.rating
                } < /Text> <
                Text style = {
                    [styles.description, {
                        color: theme.text
                    }]
                } > {
                    place.description || "No description available."
                } < /Text> <
                /View> <
                /ScrollView>
            );
        };

        const styles = StyleSheet.create({
            container: {
                flex: 1,
                backgroundColor: '#fff',
            },
            image: {
                width: '100%',
                height: 200,
            },
            details: {
                padding: 20,
                borderRadius: 8,
            },
            name: {
                fontSize: 24,
                fontWeight: 'bold',
            },
            address: {
                fontSize: 16,
                color: 'gray',
            },
            rating: {
                fontSize: 16,
            },
            description: {
                fontSize: 16,
                marginTop: 10,
            },
        });

        export default PlaceDetailsScreen;





        import React, {
            useState
        } from 'react';
        import {
            View,
            Text,
            StyleSheet,
            TextInput,
            Button,
            Alert
        } from 'react-native';
        import ProfileHeader from '../components/ProfileHeader';
        import {
            useTheme
        } from '../styles/theme'; // Import the useTheme hook

        const ProfileScreen = () => {
            const {
                theme
            } = useTheme(); // Access the theme

            const [user, setUser] = useState({
                name: 'John Doe',
                avatar: 'https://via.placeholder.com/150',
                bio: 'Travel enthusiast and explorer',
            });

            const [isEditing, setIsEditing] = useState(false);
            const [editedName, setEditedName] = useState(user.name);
            const [editedBio, setEditedBio] = useState(user.bio);

            const handleEdit = () => {
                setIsEditing(true);
            };

            const handleSave = () => {
                setUser({
                    ...user,
                    name: editedName,
                    bio: editedBio
                });
                setIsEditing(false);
                Alert.alert('Success', 'Profile updated successfully!');
            };

            return ( <
                View style = {
                    [styles.container, {
                        backgroundColor: theme.background
                    }]
                }
                accessible = {
                    true
                }
                accessibilityLabel = "Profile Screen"
                accessibilityHint = "Allows viewing and editing user profile information" >
                <
                ProfileHeader user = {
                    user
                }
                /> <
                View style = {
                    [styles.details, {
                        backgroundColor: theme.cardBackground
                    }]
                } > {
                    isEditing ? ( <
                        View accessible = {
                            true
                        }
                        accessibilityLabel = "Edit Profile Section" >
                        <
                        Text style = {
                            [styles.label, {
                                color: theme.text
                            }]
                        } > Name: < /Text> <
                        TextInput style = {
                            [styles.input, {
                                color: theme.text,
                                borderColor: theme.inputBorder
                            }]
                        }
                        value = {
                            editedName
                        }
                        onChangeText = {
                            setEditedName
                        }
                        accessible = {
                            true
                        }
                        accessibilityLabel = "Name Input"
                        accessibilityHint = "Allows editing of the user's name" /
                        >
                        <
                        Text style = {
                            [styles.label, {
                                color: theme.text
                            }]
                        } > Bio: < /Text> <
                        TextInput style = {
                            [styles.input, {
                                color: theme.text,
                                borderColor: theme.inputBorder
                            }]
                        }
                        value = {
                            editedBio
                        }
                        onChangeText = {
                            setEditedBio
                        }
                        multiline accessible = {
                            true
                        }
                        accessibilityLabel = "Bio Input"
                        accessibilityHint = "Allows editing of the user's bio" /
                        >
                        <
                        Button title = "Save"
                        onPress = {
                            handleSave
                        }
                        color = {
                            theme.primary
                        }
                        accessible = {
                            true
                        }
                        accessibilityLabel = "Save Changes"
                        accessibilityHint = "Saves the changes made to the profile" /
                        >
                        <
                        /View>
                    ) : ( <
                        View accessible = {
                            true
                        }
                        accessibilityLabel = "View Profile Section" >
                        <
                        Text style = {
                            [styles.heading, {
                                color: theme.text
                            }]
                        } > Travel History < /Text> <
                        Text style = {
                            [styles.text, {
                                color: theme.text
                            }]
                        } > No recent trips < /Text> <
                        Button title = "Edit Profile"
                        onPress = {
                            handleEdit
                        }
                        color = {
                            theme.primary
                        }
                        accessible = {
                            true
                        }
                        accessibilityLabel = "Edit Profile"
                        accessibilityHint = "Allows the user to edit their profile information" /
                        >
                        <
                        /View>
                    )
                } <
                /View> <
                /View>
            );
        };

        const styles = StyleSheet.create({
            container: {
                flex:
