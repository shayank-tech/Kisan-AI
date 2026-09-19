const ComponentFunction = function() {
// @section:imports @depends:[]
const React = require('react');
const { useState, useEffect, useRef } = React;
const {
  View, Text, StyleSheet, ScrollView, TouchableOpacity, TextInput, Platform,
  Alert, StatusBar, ActivityIndicator, Dimensions, Modal, FlatList
} = require('react-native');
const { useSafeAreaInsets } = require('react-native-safe-area-context');
const { createBottomTabNavigator } = require('@react-navigation/bottom-tabs');
const { createStackNavigator } = require('@react-navigation/stack');
const { LinearGradient } = require('expo-linear-gradient');
const { Ionicons } = require('@react-native-vector-icons/ionicons');
const { MaterialIcons } = require('@react-native-vector-icons/material-icons');
const { MaterialDesignIcons } = require('@react-native-vector-icons/material-design-icons');
const { useQuery, useMutation, useStorage } = require('platform-hooks');
const { useSpeech } = require('platform-hooks');
// @end:imports

// @section:theme @depends:[]
var PRIMARY_COLOR = '#2D5016';
var ACCENT_COLOR = '#F59E0B';
var BACKGROUND_COLOR = '#FAFAF8';
var CARD_COLOR = '#FFFFFF';
var TEXT_PRIMARY = '#1F2937';
var TEXT_SECONDARY = '#6B7280';
var VISUAL_DESIGN = 'Government-grade agri-tech command console: earthy trustworthy palette, crisp data-dense dashboards, load-indicator motif.';
var TAB_MENU_HEIGHT = Platform.OS === 'web' ? 56 : 49;
var SCROLL_EXTRA_PADDING = 16;
var WEB_TAB_MENU_PADDING = 90;
var FAB_SPACING = 16;
// @end:theme

// @section:navigation-setup @depends:[]
var Tab = createBottomTabNavigator();
var Stack = createStackNavigator();
// @end:navigation-setup

// @section:ThemeContext @depends:[theme]
var ThemeContext = React.createContext(null);
var ThemeProvider = function(props) {
  var themeValue = {
    theme: {
      colors: {
        primary: PRIMARY_COLOR,
        accent: ACCENT_COLOR,
        background: BACKGROUND_COLOR,
        card: CARD_COLOR,
        textPrimary: TEXT_PRIMARY,
        textSecondary: TEXT_SECONDARY
      }
    },
    darkMode: false,
    toggleDarkMode: function() {},
    visualDesign: VISUAL_DESIGN
  };
  return React.createElement(ThemeContext.Provider, { value: themeValue }, props.children);
};
var useTheme = function() { return React.useContext(ThemeContext); };
// @end:ThemeContext

// @section:constants @depends:[]
function genId() {
  return 'id-' + Date.now().toString(36) + '-' + Math.floor(Math.random() * 1000000).toString(36);
}
var CENTRES = [
  { id: 'centre-a', uuid: '11111111-1111-1111-1111-111111111111', name: 'Procurement Centre A', location: 'Mandya Rural', dailyCapacity: 500, counters: 4, staff: 20 },
  { id: 'centre-b', uuid: '22222222-2222-2222-2222-222222222222', name: 'Procurement Centre B', location: 'Mysuru Belt', dailyCapacity: 450, counters: 4, staff: 18 },
  { id: 'centre-c', uuid: '33333333-3333-3333-3333-333333333333', name: 'Procurement Centre C', location: 'Hassan District', dailyCapacity: 400, counters: 3, staff: 15 }
];
var CROPS = [
  { id: 'crop-paddy', uuid: 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa', name: 'Paddy', avgQuantity: 2.5 },
  { id: 'crop-wheat', uuid: 'bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb', name: 'Wheat', avgQuantity: 2.1 },
  { id: 'crop-maize', uuid: 'cccccccc-cccc-cccc-cccc-cccccccccccc', name: 'Maize', avgQuantity: 1.8 },
  { id: 'crop-other', uuid: 'dddddddd-dddd-dddd-dddd-dddddddddddd', name: 'Other', avgQuantity: 1.2 }
];
var TIME_SLOTS = ['09:00 AM', '10:30 AM', '12:00 PM', '02:00 PM', '04:00 PM'];
var WAREHOUSES = [
  { id: 'wh-a', name: 'Warehouse A', location: 'Bangalore', capacity: 5000, current: 3600, type: 'cold-storage' },
  { id: 'wh-b', name: 'Warehouse B', location: 'Mysuru', capacity: 4000, current: 1800, type: 'dry-storage' },
  { id: 'wh-c', name: 'Warehouse C', location: 'Hassan', capacity: 3000, current: 1350, type: 'processing' }
];
var VEHICLES = [
  { id: 'v1', type: 'Truck (10MT)', capacity: 10, available: 8 },
  { id: 'v2', type: 'Truck (20MT)', capacity: 20, available: 5 },
  { id: 'v3', type: 'Van (5MT)', capacity: 5, available: 12 }
];
var DEFAULT_CENTRE_STATE = {
  'centre-a': { current: 87, predicted: 94, queue: 143 },
  'centre-b': { current: 43, predicted: 51, queue: 52 },
  'centre-c': { current: 61, predicted: 68, queue: 77 }
};
var STATUS_STEPS = ['confirmed', 'arrived', 'quality_verified', 'weighing', 'completed'];
var STATUS_LABELS = {
  confirmed: 'Appointment Confirmed',
  arrived: 'Arrived at Centre',
  quality_verified: 'Quality Verified',
  weighing: 'Weighing in Progress',
  completed: 'Procurement Completed'
};
var BANK_DETAILS = {
  accountName: 'Kisan Procurement Authority',
  accountNumber: '9876543210',
  ifscCode: 'KISAN0001',
  bankName: 'State Agricultural Bank',
  branchName: 'Bangalore Main Branch'
};
function loadColor(pct) {
  if (pct >= 85) return '#DC2626';
  if (pct >= 60) return '#F59E0B';
  return '#16A34A';
}
function loadLabel(pct) {
  if (pct >= 85) return 'HIGH';
  if (pct >= 60) return 'MODERATE';
  return 'LOW';
}
function computeRecommendation(preferredCentreId, centreState) {
  var options = CENTRES.map(function(c) {
    var st = centreState[c.id] || { current: 50, predicted: 50, queue: 50 };
    return { centre: c, predicted: st.predicted, queue: st.queue };
  });
  options.sort(function(a, b) { return a.predicted - b.predicted; });
  var best = options[0];
  var preferred = options.filter(function(o) { return o.centre.id === preferredCentreId; })[0];
  var chosen = (preferred && preferred.predicted <= best.predicted + 10) ? preferred : best;
  var distance = 4 + (chosen.centre.id.charCodeAt(chosen.centre.id.length - 1) % 12);
  var waitMinutes = Math.round(chosen.queue * 0.28) + 5;
  var confidence = Math.max(60, 100 - chosen.predicted);
  var reasonCentre = preferred && preferred.centre.id !== chosen.centre.id ? preferred.centre.name : null;
  return {
    centre: chosen.centre,
    date: new Date(Date.now() + 2 * 24 * 60 * 60 * 1000).toISOString().slice(0, 10),
    time: TIME_SLOTS[Math.floor(Math.random() * TIME_SLOTS.length)],
    waitMinutes: waitMinutes,
    distanceKm: distance,
    confidence: confidence,
    reason: reasonCentre
      ? (reasonCentre + ' is predicted to reach ' + chosen.predicted + '% capacity during your preferred period. ' + chosen.centre.name + ' has lower predicted congestion.')
      : (chosen.centre.name + ' offers the lowest predicted congestion (' + chosen.predicted + '%) for your window.')
  };
}
// @end:constants

// @section:DatePickerInput @depends:[styles]
var DatePickerInput = function(props) {
  var parsed = props.value ? props.value.split('-') : null;
  var nowYear  = new Date().getFullYear();
  var nowMonth = new Date().getMonth() + 1;
  var nowDay   = new Date().getDate();
  var initYear  = parsed ? parseInt(parsed[0], 10) : nowYear;
  var initMonth = parsed ? parseInt(parsed[1], 10) : nowMonth;
  var initDay   = parsed ? parseInt(parsed[2], 10) : nowDay;

  var showState   = useState(false);
  var showPicker  = showState[0];
  var setShow     = showState[1];
  var yearState   = useState(initYear);
  var selYear     = yearState[0];
  var setSelYear  = yearState[1];
  var monthState  = useState(initMonth);
  var selMonth    = monthState[0];
  var setSelMonth = monthState[1];
  var dayState    = useState(initDay);
  var selDay      = dayState[0];
  var setSelDay   = dayState[1];

  useEffect(function() {
    var maxDay = new Date(selYear, selMonth, 0).getDate();
    if (selDay > maxDay) { setSelDay(maxDay); }
  }, [selYear, selMonth]);

  var pad = function(n) { return n < 10 ? '0' + n : String(n); };

  var handleConfirm = function() {
    if (props.onChange) {
      props.onChange(selYear + '-' + pad(selMonth) + '-' + pad(selDay));
    }
    setShow(false);
  };

  var displayValue = props.value
    ? (pad(selMonth) + '/' + pad(selDay) + '/' + selYear)
    : (props.placeholder || 'Select date');

  if (Platform.OS === 'web') {
    return React.createElement('input', {
      type: 'date',
      value: props.value || '',
      onChange: function(e) { if (props.onChange) { props.onChange(e.target.value); } },
      style: Object.assign({}, {
        padding: 12, border: '1px solid #CBD5E1', borderRadius: 8,
        fontSize: 16, width: '100%', boxSizing: 'border-box', color: '#1F2937',
        backgroundColor: '#FFFFFF', outline: 'none'
      }, props.style)
    });
  }

  var MONTHS = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
  var years  = [];
  for (var y = nowYear - 10; y <= nowYear + 20; y++) { years.push(y); }
  var daysInMonth = new Date(selYear, selMonth, 0).getDate();
  var days = [];
  for (var d = 1; d <= daysInMonth; d++) { days.push(d); }

  var itemStyle = function(active) {
    return { paddingVertical: 10, alignItems: 'center',
             backgroundColor: active ? '#EFF6FF' : 'transparent' };
  };
  var itemTextStyle = function(active) {
    return { fontSize: 16, color: active ? '#1D4ED8' : '#374151',
             fontWeight: active ? 'bold' : 'normal' };
  };
  var colStyle = { flex: 1, maxHeight: 180 };

  return React.createElement(View, { componentId: props.componentId },
    React.createElement(TouchableOpacity, {
      onPress: function() { setShow(true); },
      style: Object.assign({}, {
        flexDirection: 'row', alignItems: 'center', justifyContent: 'space-between',
        borderWidth: 1, borderColor: '#CBD5E1', borderRadius: 8,
        paddingHorizontal: 12, paddingVertical: 12, backgroundColor: '#FFFFFF'
      }, props.style)
    },
      React.createElement(Text, {
        style: { fontSize: 16, color: props.value ? '#1F2937' : '#9CA3AF' }
      }, displayValue),
      React.createElement(Text, { style: { fontSize: 18 } }, '📅')
    ),
    React.createElement(Modal, {
      visible: showPicker,
      transparent: true,
      animationType: 'slide',
      onRequestClose: function() { setShow(false); }
    },
      React.createElement(View, {
        style: { flex: 1, justifyContent: 'flex-end', backgroundColor: 'rgba(0,0,0,0.45)' }
      },
        React.createElement(View, {
          style: { backgroundColor: '#FFFFFF', borderTopLeftRadius: 20,
                   borderTopRightRadius: 20, padding: 20, paddingBottom: 36 }
        },
          React.createElement(Text, {
            style: { fontSize: 18, fontWeight: 'bold', textAlign: 'center', marginBottom: 16, color: '#111827' }
          }, 'Select Date'),
          React.createElement(View, { style: { flexDirection: 'row', borderRadius: 8, overflow: 'hidden' } },
            React.createElement(ScrollView, { style: colStyle, showsVerticalScrollIndicator: false },
              MONTHS.map(function(m, i) {
                var active = selMonth === i + 1;
                return React.createElement(TouchableOpacity, {
                  key: String(i), onPress: function() { setSelMonth(i + 1); },
                  style: itemStyle(active)
                }, React.createElement(Text, { style: itemTextStyle(active) }, m));
              })
            ),
            React.createElement(ScrollView, { style: colStyle, showsVerticalScrollIndicator: false },
              days.map(function(d) {
                var active = selDay === d;
                return React.createElement(TouchableOpacity, {
                  key: String(d), onPress: function() { setSelDay(d); },
                  style: itemStyle(active)
                }, React.createElement(Text, { style: itemTextStyle(active) }, String(d)));
              })
            ),
            React.createElement(ScrollView, { style: colStyle, showsVerticalScrollIndicator: false },
              years.map(function(yr) {
                var active = selYear === yr;
                return React.createElement(TouchableOpacity, {
                  key: String(yr), onPress: function() { setSelYear(yr); },
                  style: itemStyle(active)
                }, React.createElement(Text, { style: itemTextStyle(active) }, String(yr)));
              })
            )
          ),
          React.createElement(TouchableOpacity, {
            onPress: handleConfirm,
            style: { backgroundColor: '#1D4ED8', borderRadius: 10, padding: 14,
                     alignItems: 'center', marginTop: 16 }
          }, React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 16, fontWeight: 'bold' } }, 'Confirm')),
          React.createElement(TouchableOpacity, {
            onPress: function() { setShow(false); },
            style: { padding: 14, alignItems: 'center', marginTop: 4 }
          }, React.createElement(Text, { style: { color: '#6B7280', fontSize: 16 } }, 'Cancel'))
        )
      )
    )
  );
};
// @end:DatePickerInput

// @section:BarChartSimple @depends:[styles]
var BarChartSimple = function(props) {
  var data = props.data || [];
  var maxValue = props.maxValue || Math.max.apply(null, data.map(function(d) { return d.value; }).concat([1]));
  return React.createElement(View, { style: { flexDirection: 'row', alignItems: 'flex-end', height: 120, paddingTop: 8 }, componentId: 'bar-chart-' + (props.chartId || 'x') },
    data.map(function(d, idx) {
      var h = Math.max(4, Math.round((d.value / maxValue) * 100));
      return React.createElement(View, { key: String(idx), style: { flex: 1, alignItems: 'center', marginHorizontal: 4 } },
        React.createElement(Text, { style: { fontSize: 10, color: TEXT_SECONDARY, marginBottom: 4 } }, String(d.value)),
        React.createElement(View, { style: { width: '70%', height: h, backgroundColor: d.color || PRIMARY_COLOR, borderRadius: 6 } }),
        React.createElement(Text, { style: { fontSize: 10, color: TEXT_SECONDARY, marginTop: 6 } }, d.label)
      );
    })
  );
};
// @end:BarChartSimple

// @section:LoginScreen @depends:[theme,styles,constants]
var LoginScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  
  var nameState = useState('');
  var phoneState = useState('');
  var stageState = useState('phone'); // 'phone' | 'otp'
  var otpState = useState('');
  var loadingState = useState(false);
  var generatedOtpState = useState('');
  var userStorageState = useStorage('currentUser', null);
  var setCurrentUser = userStorageState[1];

  var handlePhoneSubmit = function() {
    if (!nameState[0].trim() || !phoneState[0].trim()) {
      Platform.OS === 'web' ? window.alert('Please enter name and phone number') : Alert.alert('Missing Info', 'Please enter name and phone number');
      return;
    }
    if (phoneState[0].replace(/[^0-9]/g, '').length < 10) {
      Platform.OS === 'web' ? window.alert('Please enter a valid 10-digit phone number') : Alert.alert('Invalid Phone', 'Please enter a valid 10-digit phone number');
      return;
    }
    loadingState[1](true);
    setTimeout(function() {
      var otp = Math.floor(100000 + Math.random() * 900000).toString();
      generatedOtpState[1](otp);
      stageState[1]('otp');
      loadingState[1](false);
      var msg = 'SMS sent with OTP: ' + otp + '\n\nFor demonstration purposes, the OTP is shown above. In production, this would be sent securely via SMS gateway to ' + phoneState[0];
      Platform.OS === 'web' ? window.alert(msg) : Alert.alert('OTP Sent', msg);
    }, 1200);
  };

  var handleOtpSubmit = function() {
    if (otpState[0].trim() !== generatedOtpState[0]) {
      Platform.OS === 'web' ? window.alert('Invalid OTP') : Alert.alert('Invalid OTP', 'Please enter the correct OTP');
      return;
    }
    // Store user in persistent storage - this keeps them logged in
    setCurrentUser({ 
      name: nameState[0], 
      phone: phoneState[0], 
      loginTime: new Date().toISOString(),
      userId: 'user-' + Date.now()
    });
    // After login, RootSwitcher will automatically show RoleSelectScreen
  };

  var ROLES = [
    { id: 'farmer', label: 'Farmer', icon: 'leaf', desc: 'Book appointments, track queue, chat with Kisan AI' },
    { id: 'officer', label: 'Procurement Officer', icon: 'clipboard-outline', desc: 'Command centre, AI recommendations, load balancing' },
    { id: 'admin', label: 'Administrator', icon: 'shield-checkmark-outline', desc: 'National overview, analytics, audit logs' }
  ];

  if (stageState[0] === 'phone') {
    return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
      React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
      React.createElement(LinearGradient, {
        colors: [PRIMARY_COLOR, '#1A3009'],
        style: { paddingTop: insets.top + 40, paddingBottom: 32, paddingHorizontal: 24, borderBottomLeftRadius: 24, borderBottomRightRadius: 24 }
      },
        React.createElement(MaterialDesignIcons, { name: 'sprout', size: 40, color: ACCENT_COLOR }),
        React.createElement(Text, { style: { fontSize: 26, fontWeight: '800', color: '#FFFFFF', marginTop: 12 } }, 'KisanProcure AI'),
        React.createElement(Text, { style: { fontSize: 14, color: '#D9E5C8', marginTop: 6 } }, 'Autonomous Procurement Orchestration Platform')
      ),
      React.createElement(ScrollView, { contentContainerStyle: { padding: 20, paddingBottom: insets.bottom + SCROLL_EXTRA_PADDING } },
        React.createElement(Text, { style: { fontSize: 18, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 20 } }, 'Login to Your Account'),
        React.createElement(Text, { style: styles.label }, 'Full Name'),
        React.createElement(TextInput, {
          style: styles.input,
          value: nameState[0],
          onChangeText: nameState[1],
          placeholder: 'Enter your full name',
          autoCapitalize: 'words',
          componentId: 'login-name-input'
        }),
        React.createElement(Text, { style: styles.label }, 'Phone Number'),
        React.createElement(TextInput, {
          style: styles.input,
          value: phoneState[0],
          onChangeText: function(t) { phoneState[1](t.replace(/[^0-9+\-() ]/g, '')); },
          placeholder: '+91 XXXXX XXXXX',
          keyboardType: 'phone-pad',
          componentId: 'login-phone-input'
        }),
        React.createElement(TouchableOpacity, {
          style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 20 },
          onPress: handlePhoneSubmit,
          componentId: 'login-submit-phone'
        },
          loadingState[0]
            ? React.createElement(ActivityIndicator, { color: '#1F2937' })
            : React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800', fontSize: 15 } }, 'SEND OTP')
        ),
        React.createElement(View, { style: { backgroundColor: ACCENT_COLOR + '1A', borderRadius: 12, padding: 14, marginTop: 20 } },
          React.createElement(Text, { style: { fontSize: 12, color: '#92600A', fontWeight: '600' } }, 'Demo Mode: OTP will be displayed after submission. In production, SMS would be sent to your registered number.')
        )
      )
    );
  }

  if (stageState[0] === 'otp') {
    return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
      React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
      React.createElement(LinearGradient, {
        colors: [PRIMARY_COLOR, '#1A3009'],
        style: { paddingTop: insets.top + 40, paddingBottom: 32, paddingHorizontal: 24, borderBottomLeftRadius: 24, borderBottomRightRadius: 24 }
      },
        React.createElement(MaterialDesignIcons, { name: 'sprout', size: 40, color: ACCENT_COLOR }),
        React.createElement(Text, { style: { fontSize: 26, fontWeight: '800', color: '#FFFFFF', marginTop: 12 } }, 'Verify OTP')
      ),
      React.createElement(ScrollView, { contentContainerStyle: { padding: 20, paddingBottom: insets.bottom + SCROLL_EXTRA_PADDING } },
        React.createElement(Text, { style: { fontSize: 16, color: theme.colors.textPrimary, marginBottom: 8 } }, 'Enter the 6-digit code sent to'),
        React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: PRIMARY_COLOR, marginBottom: 20 } }, phoneState[0]),
        React.createElement(Text, { style: styles.label }, 'OTP Code'),
        React.createElement(TextInput, {
          style: [styles.input, { fontSize: 24, letterSpacing: 8, textAlign: 'center', fontWeight: '700' }],
          value: otpState[0],
          onChangeText: function(t) { otpState[1](t.replace(/[^0-9]/g, '').slice(0, 6)); },
          placeholder: '000000',
          keyboardType: 'numeric',
          maxLength: 6,
          componentId: 'otp-input'
        }),
        React.createElement(TouchableOpacity, {
          style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 20 },
          onPress: handleOtpSubmit,
          componentId: 'otp-submit'
        },
          React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800', fontSize: 15 } }, 'VERIFY OTP')
        ),
        React.createElement(TouchableOpacity, {
          style: { marginTop: 14, alignItems: 'center' },
          onPress: function() { stageState[1]('phone'); otpState[1](''); },
          componentId: 'otp-back'
        },
          React.createElement(Text, { style: { color: PRIMARY_COLOR, fontWeight: '700' } }, 'Change Phone Number')
        ),
        React.createElement(View, { style: { backgroundColor: ACCENT_COLOR + '1A', borderRadius: 12, padding: 14, marginTop: 20 } },
          React.createElement(Text, { style: { fontSize: 12, color: '#92600A', fontWeight: '600' } }, 'Demo OTP: ' + generatedOtpState[0])
        )
      )
    );
  }
};
// @end:LoginScreen

// @section:RoleSelectScreen @depends:[ThemeContext,styles]
var RoleSelectScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var roleState = useStorage('activeRole', null);
  var setRole = roleState[1];
  var currentUserStorage = useStorage('currentUser', null);
  var currentUser = currentUserStorage[0];
  var setCurrentUser = currentUserStorage[1];

  var handleLogout = function() {
    setCurrentUser(null);
    setRole(null);
  };

  var ROLES = [
    { id: 'farmer', label: 'Farmer', icon: 'leaf', desc: 'Book appointments, track queue, chat with Kisan AI' },
    { id: 'officer', label: 'Procurement Officer', icon: 'clipboard-outline', desc: 'Command centre, AI recommendations, load balancing' },
    { id: 'admin', label: 'Administrator', icon: 'shield-checkmark-outline', desc: 'National overview, analytics, audit logs' }
  ];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(LinearGradient, {
      colors: [PRIMARY_COLOR, '#1A3009'],
      style: { paddingTop: insets.top + 40, paddingBottom: 32, paddingHorizontal: 24, borderBottomLeftRadius: 24, borderBottomRightRadius: 24, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'flex-start' }
    },
      React.createElement(View, null,
        React.createElement(MaterialDesignIcons, { name: 'sprout', size: 40, color: ACCENT_COLOR }),
        React.createElement(Text, { style: { fontSize: 26, fontWeight: '800', color: '#FFFFFF', marginTop: 12 } }, 'KisanProcure AI'),
        React.createElement(Text, { style: { fontSize: 14, color: '#D9E5C8', marginTop: 6 } }, 'Autonomous Procurement Orchestration Platform')
      ),
      React.createElement(TouchableOpacity, { onPress: handleLogout, componentId: 'role-select-logout' },
        React.createElement(Ionicons, { name: 'log-out', size: 22, color: '#FFFFFF' })
      )
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 20, paddingBottom: insets.bottom + SCROLL_EXTRA_PADDING } },
      React.createElement(Text, { style: { fontSize: 16, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 6 } }, 'Welcome, ' + (currentUser ? currentUser.name.split(' ')[0] : 'User')),
      React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary, marginBottom: 16 } }, 'Select your role to continue'),
      ROLES.map(function(r) {
        return React.createElement(TouchableOpacity, {
          key: r.id,
          onPress: function() { setRole(r.id); },
          style: { backgroundColor: theme.colors.card, borderRadius: 16, padding: 18, marginBottom: 14, flexDirection: 'row', alignItems: 'center', shadowColor: '#000', shadowOpacity: 0.08, shadowRadius: 6, shadowOffset: { width: 0, height: 2 }, elevation: 2 },
          componentId: 'role-select-' + r.id
        },
          React.createElement(View, { style: { width: 48, height: 48, borderRadius: 24, backgroundColor: PRIMARY_COLOR + '1A', alignItems: 'center', justifyContent: 'center', marginRight: 14 } },
            React.createElement(Ionicons, { name: r.icon, size: 24, color: PRIMARY_COLOR })
          ),
          React.createElement(View, { style: { flex: 1 } },
            React.createElement(Text, { style: { fontSize: 16, fontWeight: '700', color: theme.colors.textPrimary } }, r.label),
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 2 } }, r.desc)
          ),
          React.createElement(Ionicons, { name: 'chevron-forward', size: 20, color: theme.colors.textSecondary })
        );
      }),
      React.createElement(View, { style: { backgroundColor: ACCENT_COLOR + '1A', borderRadius: 12, padding: 14, marginTop: 12 } },
        React.createElement(Text, { style: { fontSize: 12, color: '#92600A', fontWeight: '600' } }, 'Logged in as: ' + (currentUser ? currentUser.phone : 'Unknown') + ' · SIH DEMO MODE')
      )
    )
  );
};
// @end:RoleSelectScreen

// @section:FarmerDashboardScreen-state @depends:[]
var DEFAULT_FARMER = { name: '', phone: '', farmerId: '', location: '' };
var DEFAULT_APPOINTMENT = null;
// @end:FarmerDashboardScreen-state

// @section:FarmerDashboardScreen @depends:[ThemeContext,FarmerDashboardScreen-state,styles]
var FarmerDashboardScreen = function(props) {
  var navigation = props.navigation;
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var appointmentState = useStorage('farmerAppointment', DEFAULT_APPOINTMENT);
  var appointment = appointmentState[0];
  var roleState = useStorage('activeRole', null);
  var setRole = roleState[1];
  var currentUserStorage = useStorage('currentUser', null);
  var currentUser = currentUserStorage[0];

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var currentIndex = appointment ? STATUS_STEPS.indexOf(appointment.status) : -1;

  var handleSwitchRole = function() {
    setRole(null);
    // User stays logged in, just role is cleared - will show RoleSelectScreen
  };

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(LinearGradient, {
      colors: [PRIMARY_COLOR, '#3E6B22'],
      style: { paddingTop: insets.top + 16, paddingBottom: 20, paddingHorizontal: 20, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' }
    },
      React.createElement(View, null,
        React.createElement(Text, { style: { color: '#D9E5C8', fontSize: 13 } }, 'Welcome, Farmer'),
        React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 20, fontWeight: '800' } }, 'My Procurement')
      ),
      React.createElement(TouchableOpacity, { onPress: handleSwitchRole, componentId: 'farmer-switch-role' },
        React.createElement(Ionicons, { name: 'swap-horizontal', size: 22, color: '#FFFFFF' })
      )
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      !appointment ? React.createElement(View, { style: styles.card },
        React.createElement(Ionicons, { name: 'leaf-outline', size: 32, color: PRIMARY_COLOR }),
        React.createElement(Text, { style: { fontSize: 16, fontWeight: '700', color: theme.colors.textPrimary, marginTop: 10 } }, 'No active procurement'),
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary, marginTop: 4 } }, 'Book an appointment to get an AI-optimized procurement slot.'),
        React.createElement(TouchableOpacity, {
          style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 14, alignItems: 'center', marginTop: 14 },
          onPress: function() { navigation.navigate('Book'); },
          componentId: 'farmer-book-cta'
        }, React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800' } }, 'BOOK APPOINTMENT'))
      ) : React.createElement(View, null,
        React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap', marginHorizontal: -6 } },
          [
            { icon: 'leaf', label: 'Active Procurement', value: appointment.cropName + ' · ' + appointment.quantity + 't' },
            { icon: 'ticket', label: 'Token', value: appointment.token },
            { icon: 'location', label: 'Centre', value: appointment.centreName },
            { icon: 'time', label: 'Expected Waiting', value: appointment.waitMinutes + ' min' },
            { icon: 'people', label: 'Queue Position', value: String(appointment.queuePosition) }
          ].map(function(k, idx) {
            return React.createElement(View, { key: String(idx), style: { width: '50%', padding: 6 } },
              React.createElement(View, { style: [styles.card, { padding: 14 }] },
                React.createElement(Ionicons, { name: k.icon, size: 20, color: ACCENT_COLOR }),
                React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 6 } }, k.label),
                React.createElement(Text, { style: { fontSize: 15, fontWeight: '800', color: theme.colors.textPrimary, marginTop: 2 } }, k.value)
              )
            );
          })
        ),
        React.createElement(View, { style: [styles.card, { marginTop: 6 }] },
          React.createElement(Text, { style: { fontSize: 15, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 12 } }, 'Status Timeline'),
          STATUS_STEPS.map(function(s, idx) {
            var done = idx <= currentIndex;
            return React.createElement(View, { key: s, style: { flexDirection: 'row', alignItems: 'center', marginBottom: idx === STATUS_STEPS.length - 1 ? 0 : 10 } },
              React.createElement(View, { style: { width: 22, height: 22, borderRadius: 11, backgroundColor: done ? PRIMARY_COLOR : '#E5E7EB', alignItems: 'center', justifyContent: 'center', marginRight: 10 } },
                done ? React.createElement(Ionicons, { name: 'checkmark', size: 13, color: '#FFFFFF' }) : null
              ),
              React.createElement(Text, { style: { fontSize: 13, color: done ? theme.colors.textPrimary : theme.colors.textSecondary, fontWeight: done ? '700' : '400' } }, STATUS_LABELS[s])
            );
          })
        ),
        React.createElement(TouchableOpacity, {
          style: { backgroundColor: PRIMARY_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 16 },
          onPress: function() { navigation.navigate('Track'); },
          componentId: 'farmer-track-cta'
        }, React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '800', fontSize: 15 } }, 'TRACK PROCUREMENT'))
      )
    )
  );
};
// @end:FarmerDashboardScreen

// @section:BookAppointmentScreen @depends:[ThemeContext,constants,DatePickerInput,styles]
var BookAppointmentScreen = function(props) {
  var navigation = props.navigation;
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();

  var nameState = useState('');
  var phoneState = useState('');
  var locationState = useState('');
  var quantityState = useState('');
  var harvestDateState = useState('');
  var cropIdState = useState(CROPS[0].id);
  var centreIdState = useState(CENTRES[0].id);
  var timeSlotState = useState(TIME_SLOTS[0]);
  var stageState = useState('form');
  var recoState = useState(null);
  var submittingState = useState(false);

  var centreState = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreState[0];
  var appointmentStorage = useStorage('farmerAppointment', DEFAULT_APPOINTMENT);
  var setAppointment = appointmentStorage[1];

  var { mutate: insertAppointment } = useMutation('appointments', 'insert');
  var { mutate: insertNotification } = useMutation('notifications', 'insert');

  var handleSubmit = function() {
    if (!nameState[0] || !phoneState[0] || !locationState[0] || !quantityState[0] || !harvestDateState[0]) {
      Platform.OS === 'web' ? window.alert('Please fill all required fields') : Alert.alert('Missing Info', 'Please fill all required fields');
      return;
    }
    submittingState[1](true);
    setTimeout(function() {
      var reco = computeRecommendation(centreIdState[0], centreLoads);
      recoState[1](reco);
      stageState[1]('reco');
      submittingState[1](false);
    }, 1400);
  };

  var acceptReco = function() {
    var reco = recoState[0];
    var crop = CROPS.filter(function(c) { return c.id === cropIdState[0]; })[0];
    var token = 'KSN' + Math.floor(1000 + Math.random() * 8999);
    var newAppointment = {
      centreId: reco.centre.id,
      centreName: reco.centre.name,
      cropName: crop.name,
      quantity: quantityState[0],
      date: reco.date,
      time: reco.time,
      token: token,
      waitMinutes: reco.waitMinutes,
      queuePosition: (centreLoads[reco.centre.id] || {}).queue || 40,
      distanceKm: reco.distanceKm,
      confidence: reco.confidence,
      status: 'confirmed'
    };
    setAppointment(newAppointment);
    insertAppointment({
      farmer_id: genId(),
      centre_id: reco.centre.uuid,
      crop_id: crop.uuid,
      appointment_date: reco.date,
      appointment_time: reco.time,
      quantity_tonnes: parseFloat(quantityState[0]) || crop.avgQuantity,
      harvest_date: harvestDateState[0],
      status: 'confirmed',
      token_number: token,
      predicted_waiting_minutes: reco.waitMinutes,
      travel_distance_km: reco.distanceKm,
      confidence_score: reco.confidence
    }).catch(function() {});
    insertNotification({
      farmer_id: genId(),
      notification_type: 'appointment_confirmed',
      title: 'Appointment Confirmed',
      message: 'Your token is ' + token + '. Expected waiting ~' + reco.waitMinutes + ' minutes at ' + reco.centre.name + '.',
      is_read: false
    }).catch(function() {});
    stageState[1]('form');
    recoState[1](null);
    navigation.navigate('Dashboard');
  };

  var scrollBottomPadding = insets.bottom + SCROLL_EXTRA_PADDING;

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Book Appointment')
    ),
    stageState[0] === 'form' ? React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(Text, { style: styles.label }, 'Farmer Name'),
      React.createElement(TextInput, { style: styles.input, value: nameState[0], onChangeText: nameState[1], placeholder: 'Full name', autoCapitalize: 'words', componentId: 'input-farmer-name' }),
      React.createElement(Text, { style: styles.label }, 'Mobile Number'),
      React.createElement(TextInput, { style: styles.input, value: phoneState[0], onChangeText: function(t) { phoneState[1](t.replace(/[^0-9+\-() ]/g, '')); }, placeholder: 'Phone number', keyboardType: 'phone-pad', componentId: 'input-phone' }),
      React.createElement(Text, { style: styles.label }, 'Location'),
      React.createElement(TextInput, { style: styles.input, value: locationState[0], onChangeText: locationState[1], placeholder: 'Village / Taluk', componentId: 'input-location' }),
      React.createElement(Text, { style: styles.label }, 'Crop'),
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap' } },
        CROPS.map(function(c) {
          var active = cropIdState[0] === c.id;
          return React.createElement(TouchableOpacity, {
            key: c.id, onPress: function() { cropIdState[1](c.id); },
            style: { paddingVertical: 8, paddingHorizontal: 14, borderRadius: 20, backgroundColor: active ? PRIMARY_COLOR : '#EEF2E9', marginRight: 8, marginBottom: 8 },
            componentId: 'crop-chip-' + c.id
          }, React.createElement(Text, { style: { color: active ? '#FFFFFF' : theme.colors.textPrimary, fontWeight: '600', fontSize: 13 } }, c.name));
        })
      ),
      React.createElement(Text, { style: styles.label }, 'Quantity (tonnes)'),
      React.createElement(TextInput, { style: styles.input, value: quantityState[0], onChangeText: function(t) { quantityState[1](t.replace(/[^0-9.]/g, '')); }, placeholder: '2.5', keyboardType: 'decimal-pad', componentId: 'input-quantity' }),
      React.createElement(Text, { style: styles.label }, 'Harvest Date'),
      React.createElement(DatePickerInput, { value: harvestDateState[0], onChange: harvestDateState[1], placeholder: 'Select harvest date', componentId: 'input-harvest-date' }),
      React.createElement(Text, { style: styles.label }, 'Preferred Centre'),
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap' } },
        CENTRES.map(function(c) {
          var active = centreIdState[0] === c.id;
          return React.createElement(TouchableOpacity, {
            key: c.id, onPress: function() { centreIdState[1](c.id); },
            style: { paddingVertical: 8, paddingHorizontal: 14, borderRadius: 20, backgroundColor: active ? PRIMARY_COLOR : '#EEF2E9', marginRight: 8, marginBottom: 8 },
            componentId: 'centre-chip-' + c.id
          }, React.createElement(Text, { style: { color: active ? '#FFFFFF' : theme.colors.textPrimary, fontWeight: '600', fontSize: 13 } }, c.name.replace('Procurement Centre ', 'Centre ')));
        })
      ),
      React.createElement(Text, { style: styles.label }, 'Preferred Time'),
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap' } },
        TIME_SLOTS.map(function(t) {
          var active = timeSlotState[0] === t;
          return React.createElement(TouchableOpacity, {
            key: t, onPress: function() { timeSlotState[1](t); },
            style: { paddingVertical: 8, paddingHorizontal: 12, borderRadius: 20, backgroundColor: active ? PRIMARY_COLOR : '#EEF2E9', marginRight: 8, marginBottom: 8 },
            componentId: 'time-chip-' + t
          }, React.createElement(Text, { style: { color: active ? '#FFFFFF' : theme.colors.textPrimary, fontWeight: '600', fontSize: 12 } }, t));
        })
      ),
      React.createElement(TouchableOpacity, {
        style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 12 },
        onPress: handleSubmit,
        componentId: 'submit-booking'
      }, submittingState[0]
        ? React.createElement(ActivityIndicator, { color: '#1F2937' })
        : React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800', fontSize: 15 } }, 'FIND OPTIMAL SLOT')
      )
    ) : React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: [styles.card, { alignItems: 'center', paddingVertical: 20 }] },
        React.createElement(MaterialDesignIcons, { name: 'robot-outline', size: 36, color: PRIMARY_COLOR }),
        React.createElement(Text, { style: { fontSize: 16, fontWeight: '800', color: theme.colors.textPrimary, marginTop: 10 } }, 'AI found your optimal slot'),
        React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4, textAlign: 'center' } }, 'Simulated/Synthetic optimization result')
      ),
      recoState[0] ? React.createElement(View, { style: [styles.card, { marginTop: 12 }] },
        [
          { label: 'Recommended Centre', value: recoState[0].centre.name },
          { label: 'Date', value: recoState[0].date },
          { label: 'Time', value: recoState[0].time },
          { label: 'Expected Waiting', value: recoState[0].waitMinutes + ' minutes' },
          { label: 'Travel Distance', value: recoState[0].distanceKm + ' km' },
          { label: 'Confidence', value: recoState[0].confidence + '%' }
        ].map(function(r, idx) {
          return React.createElement(View, { key: String(idx), style: { flexDirection: 'row', justifyContent: 'space-between', paddingVertical: 8, borderBottomWidth: idx === 5 ? 0 : 1, borderBottomColor: '#F0F0EE' } },
            React.createElement(Text, { style: { color: theme.colors.textSecondary, fontSize: 13 } }, r.label),
            React.createElement(Text, { style: { color: theme.colors.textPrimary, fontSize: 13, fontWeight: '700' } }, r.value)
          );
        }),
        React.createElement(View, { style: { backgroundColor: '#EEF2E9', borderRadius: 10, padding: 12, marginTop: 10 } },
          React.createElement(Text, { style: { fontSize: 12, color: '#33501B', lineHeight: 18 } }, recoState[0].reason)
        )
      ) : null,
      React.createElement(TouchableOpacity, { style: { backgroundColor: PRIMARY_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 16 }, onPress: acceptReco, componentId: 'accept-reco' },
        React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '800' } }, 'ACCEPT RECOMMENDATION')
      ),
      React.createElement(TouchableOpacity, { style: { borderWidth: 1, borderColor: PRIMARY_COLOR, borderRadius: 10, padding: 16, alignItems: 'center', marginTop: 10 }, onPress: function() { stageState[1]('form'); }, componentId: 'choose-manually' },
        React.createElement(Text, { style: { color: PRIMARY_COLOR, fontWeight: '800' } }, 'CHOOSE MANUALLY')
      )
    )
  );
};
// @end:BookAppointmentScreen

// @section:TrackTokenScreen @depends:[ThemeContext,styles]
var TrackTokenScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var appointmentStorage = useStorage('farmerAppointment', DEFAULT_APPOINTMENT);
  var appointment = appointmentStorage[0];
  var setAppointment = appointmentStorage[1];

  useEffect(function() {
    if (!appointment || appointment.queuePosition <= 0) return;
    var timer = setInterval(function() {
      setAppointment(function(prev) {
        if (!prev || prev.queuePosition <= 0) return prev;
        var nextPos = Math.max(0, prev.queuePosition - 1);
        var nextWait = Math.max(0, prev.waitMinutes - 2);
        var nextStatus = prev.status;
        if (nextPos === 0) nextStatus = 'completed';
        else if (nextPos <= 3) nextStatus = 'weighing';
        else if (nextPos <= 8) nextStatus = 'quality_verified';
        else if (nextPos <= prev.queuePosition && prev.status === 'confirmed') nextStatus = 'arrived';
        return Object.assign({}, prev, { queuePosition: nextPos, waitMinutes: nextWait, status: nextStatus });
      });
    }, 3000);
    return function() { clearInterval(timer); };
  }, [appointment ? appointment.token : null]);

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Track Token')
    ),
    !appointment ? React.createElement(View, { style: { flex: 1, alignItems: 'center', justifyContent: 'center', padding: 24 } },
      React.createElement(Ionicons, { name: 'ticket-outline', size: 40, color: theme.colors.textSecondary }),
      React.createElement(Text, { style: { color: theme.colors.textSecondary, marginTop: 10 } }, 'No active token to track')
    ) : React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(LinearGradient, { colors: [PRIMARY_COLOR, '#3E6B22'], style: { borderRadius: 16, padding: 20, alignItems: 'center' } },
        React.createElement(Text, { style: { color: '#D9E5C8', fontSize: 12 } }, 'Your Token'),
        React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 32, fontWeight: '900', letterSpacing: 2 } }, appointment.token)
      ),
      React.createElement(View, { style: { flexDirection: 'row', marginTop: 14 } },
        React.createElement(View, { style: [styles.card, { flex: 1, marginRight: 6, alignItems: 'center' }] },
          React.createElement(Text, { style: { fontSize: 28, fontWeight: '900', color: ACCENT_COLOR } }, String(appointment.queuePosition)),
          React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary } }, 'Queue Position')
        ),
        React.createElement(View, { style: [styles.card, { flex: 1, marginLeft: 6, alignItems: 'center' }] },
          React.createElement(Text, { style: { fontSize: 28, fontWeight: '900', color: PRIMARY_COLOR } }, String(appointment.waitMinutes)),
          React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary } }, 'Minutes Waiting')
        )
      ),
      React.createElement(View, { style: [styles.card, { marginTop: 12 }] },
        React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 8 } }, 'Status'),
        React.createElement(Text, { style: { fontSize: 16, fontWeight: '800', color: PRIMARY_COLOR } }, STATUS_LABELS[appointment.status] || appointment.status),
        React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 6 } }, appointment.centreName + ' · ' + appointment.date + ' · ' + appointment.time)
      ),
      React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, textAlign: 'center', marginTop: 14 } }, 'Live queue position updates automatically (simulated real-time feed)')
    )
  );
};
// @end:TrackTokenScreen

// @section:PaymentScreen @depends:[ThemeContext,styles]
var PaymentScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var { data: transactions, loading, refetch } = useQuery('transactions', {}, { column: 'created_at', ascending: false });
  var { mutate: updateTransaction } = useMutation('transactions', 'update');

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var list = transactions && transactions.length > 0 ? transactions : [];

  var pendingPayments = list.filter(function(t) { return t.transaction_status !== 'completed'; });
  var completedPayments = list.filter(function(t) { return t.transaction_status === 'completed'; });

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Payments')
    ),
    loading ? React.createElement(ActivityIndicator, { style: { flex: 1 }, componentId: 'payment-loading' }) :
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      pendingPayments.length > 0 ? React.createElement(View, null,
        React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 10 } }, 'Pending Payments (' + pendingPayments.length + ')'),
        pendingPayments.map(function(t) {
          var refNumber = 'KSN-' + t.id.substring(0, 8).toUpperCase();
          return React.createElement(View, { key: t.id, style: [styles.card, { marginBottom: 12, borderLeftWidth: 4, borderLeftColor: ACCENT_COLOR }] },
            React.createElement(View, { style: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'flex-start' } },
              React.createElement(View, { style: { flex: 1 } },
                React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary } }, '₹' + (t.payment_amount || '2500')),
                React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, 'Ref: ' + refNumber),
                React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, 'Quantity: ' + t.quantity_tonnes + ' tonnes')
              ),
              React.createElement(View, { style: { backgroundColor: ACCENT_COLOR + '33', paddingHorizontal: 10, paddingVertical: 4, borderRadius: 6 } },
                React.createElement(Text, { style: { fontSize: 11, fontWeight: '700', color: '#92600A' } }, 'PENDING')
              )
            ),
            React.createElement(View, { style: { backgroundColor: '#FEF3E2', borderRadius: 10, padding: 12, marginTop: 12 } },
              React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: '#92600A', marginBottom: 8 } }, 'Bank Transfer Details'),
              React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00', marginBottom: 4 } }, 'Account: ' + BANK_DETAILS.accountNumber),
              React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00', marginBottom: 4 } }, 'Bank: ' + BANK_DETAILS.bankName),
              React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00', marginBottom: 4 } }, 'IFSC: ' + BANK_DETAILS.ifscCode),
              React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00' } }, 'Reference: ' + refNumber)
            ),
            React.createElement(TouchableOpacity, {
              style: { backgroundColor: PRIMARY_COLOR, borderRadius: 8, padding: 10, alignItems: 'center', marginTop: 12 },
              onPress: function() { updateTransaction({ id: t.id, data: { transaction_status: 'completed', completed_at: new Date().toISOString() } }).then(refetch).catch(function() {}); },
              componentId: 'confirm-payment-' + t.id
            },
              React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '700', fontSize: 12 } }, 'CONFIRM PAYMENT SENT')
            )
          );
        })
      ) : React.createElement(View, { style: { alignItems: 'center', paddingVertical: 24 } },
        React.createElement(Ionicons, { name: 'checkmark-circle', size: 40, color: '#16A34A' }),
        React.createElement(Text, { style: { color: theme.colors.textSecondary, marginTop: 8 } }, 'No pending payments')
      ),
      completedPayments.length > 0 ? React.createElement(View, { style: { marginTop: 20 } },
        React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 10 } }, 'Payment History (' + completedPayments.length + ')'),
        completedPayments.map(function(t) {
          var refNumber = 'KSN-' + t.id.substring(0, 8).toUpperCase();
          var completedDate = t.completed_at ? new Date(t.completed_at).toLocaleDateString() : 'N/A';
          return React.createElement(View, { key: t.id, style: [styles.card, { marginBottom: 10, opacity: 0.8, borderLeftWidth: 4, borderLeftColor: '#16A34A' }] },
            React.createElement(View, { style: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
              React.createElement(View, { style: { flex: 1 } },
                React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary } }, '₹' + (t.payment_amount || '2500')),
                React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, completedDate + ' · ' + refNumber)
              ),
              React.createElement(View, { style: { backgroundColor: '#D1F3D8', paddingHorizontal: 10, paddingVertical: 4, borderRadius: 6 } },
                React.createElement(Text, { style: { fontSize: 11, fontWeight: '700', color: '#166534' } }, 'COMPLETED')
              )
            )
          );
        })
      ) : null
    )
  );
};
// @end:PaymentScreen

// @section:NotificationsScreen @depends:[ThemeContext,styles]
var NotificationsScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var { data: notifications, loading, refetch } = useQuery('notifications', {}, { column: 'created_at', ascending: false });
  var { mutate: updateNotification } = useMutation('notifications', 'update');

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var list = notifications || [];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Notifications')
    ),
    loading ? React.createElement(ActivityIndicator, { style: { flex: 1 }, componentId: 'notif-loading' }) :
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      list.length === 0 ? React.createElement(Text, { style: { color: theme.colors.textSecondary, textAlign: 'center', marginTop: 40 } }, 'No notifications yet') :
      list.map(function(n) {
        return React.createElement(TouchableOpacity, {
          key: n.id,
          onPress: function() { updateNotification({ id: n.id, data: { is_read: true } }).then(refetch); },
          style: [styles.card, { marginBottom: 10, opacity: n.is_read ? 0.6 : 1, borderLeftWidth: 4, borderLeftColor: n.is_read ? '#D1D5DB' : ACCENT_COLOR }],
          componentId: 'notification-' + n.id
        },
          React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary } }, n.title),
          React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary, marginTop: 4 } }, n.message)
        );
      })
    )
  );
};
// @end:NotificationsScreen

// @section:EnhancedKisanAIScreen @depends:[theme,styles,constants,useSpeech]
var EnhancedKisanAIScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var appointmentStorage = useStorage('farmerAppointment', DEFAULT_APPOINTMENT);
  var appointment = appointmentStorage[0];
  var currentUserStorage = useStorage('currentUser', null);
  var currentUser = currentUserStorage[0];
  
  var langState = useState('en');
  var inputState = useState('');
  var messagesState = useState([
    { id: 'm0', from: 'ai', text: 'Namaste! Ask me about your token, queue, or waiting time. I can also translate responses.' }
  ]);
  var translatorLangState = useState('en');
  var showTranslatorState = useState(false);
  var searchState = useState('');
  var speechHook = useSpeech();

  var LANGUAGES = [
    { id: 'en', label: 'English', nativeName: 'English' },
    { id: 'hi', label: 'हिंदी', nativeName: 'Hindi' },
    { id: 'kn', label: 'ಕನ್ನಡ', nativeName: 'Kannada' },
    { id: 'ta', label: 'தமிழ்', nativeName: 'Tamil' },
    { id: 'te', label: 'తెలుగు', nativeName: 'Telugu' },
    { id: 'ml', label: 'മലയാളം', nativeName: 'Malayalam' },
    { id: 'mr', label: 'मराठी', nativeName: 'Marathi' },
    { id: 'gu', label: 'ગુજરાતી', nativeName: 'Gujarati' },
    { id: 'bn', label: 'বাংলা', nativeName: 'Bengali' },
    { id: 'pa', label: 'ਪੰਜਾਬੀ', nativeName: 'Punjabi' }
  ];

  var translateText = function(text, targetLang) {
    if (targetLang === 'en') return text;
    var translations = {
      'hi': {
        'Your token is': 'आपका टोकन है',
        'Your queue position is': 'आपकी कतार की स्थिति है',
        'Expected waiting time is about': 'अनुमानित प्रतीक्षा समय लगभग है',
        'minutes': 'मिनट',
        'Your procurement centre is': 'आपका खरीद केंद्र है',
        'No active appointment yet': 'अभी कोई सक्रिय नियुक्ति नहीं है',
        'Please book a procurement slot first': 'कृपया पहले एक खरीद स्लॉट बुक करें'
      },
      'kn': {
        'Your token is': 'ನಿಮ್ಮ ಟೋಕನ್',
        'Your queue position is': 'ನಿಮ್ಮ ಸರದಿ ಸ್ಥಾನ',
        'Expected waiting time is about': 'ಅನುಮಾನಿತ ಪ್ರತೀಕ್ಷಾ ಸಮಯ ಸುಮಾರು',
        'minutes': 'ನಿಮಿಷಗಳು',
        'Your procurement centre is': 'ನಿಮ್ಮ ಖರೀದಿ ಕೇಂದ್ರ',
        'No active appointment yet': 'ಇನ್ನೂ ಸಕ್ರಿಯ ನಿಯೋಜನೆ ಇಲ್ಲ',
        'Please book a procurement slot first': 'ದಯವಿಟ್ಟು ಮೊದಲು ಖರೀದಿ ಸ್ಲಾಟ್ ಬುಕ್ ಮಾಡಿ'
      },
      'ta': {
        'Your token is': 'உங்கள் டோக்கன்',
        'Your queue position is': 'உங்கள் வரிசை நிலை',
        'Expected waiting time is about': 'எதிர்பார்க்கப்படும் காத்திருக்கும் நேரம் சுமார்',
        'minutes': 'நிமிடங்கள்',
        'Your procurement centre is': 'உங்கள் கொள்முதல் மையம்',
        'No active appointment yet': 'இன்னும் செயல்பாட்டு நியமனம் இல்லை',
        'Please book a procurement slot first': 'முதலில் கொள்முதல் இடத்தை பதிவு செய்யவும்'
      }
    };
    if (translations[targetLang]) {
      var result = text;
      Object.keys(translations[targetLang]).forEach(function(key) {
        result = result.replace(new RegExp(key, 'gi'), translations[targetLang][key]);
      });
      return result;
    }
    return text;
  };

  var respond = function(question) {
    var lower = question.toLowerCase();
    var text;
    if (!appointment) {
      text = 'You have no active appointment yet. Please book a procurement slot first.';
    } else if (lower.indexOf('token') > -1 || lower.indexOf('टोकन') > -1 || lower.indexOf('ಟೋಕನ್') > -1) {
      text = 'Your token is ' + appointment.token + '.';
    } else if (lower.indexOf('wait') > -1 || lower.indexOf('turn') > -1 || lower.indexOf('सरदी') > -1 || lower.indexOf('ಸರದಿ') > -1 || lower.indexOf('बारी') > -1) {
      text = 'Your queue position is ' + appointment.queuePosition + '. Expected waiting time is about ' + appointment.waitMinutes + ' minutes.';
    } else if (lower.indexOf('centre') > -1 || lower.indexOf('center') > -1) {
      text = 'Your procurement centre is ' + appointment.centreName + '.';
    } else if (lower.indexOf('status') > -1) {
      text = 'Your current status: ' + (STATUS_LABELS[appointment.status] || appointment.status) + ' at ' + appointment.centreName + '.';
    } else {
      text = 'I can help you with your token, queue position, waiting time, and centre details. What would you like to know?';
    }
    return text;
  };

  var sendMessage = function() {
    var q = inputState[0].trim();
    if (!q) return;
    var reply = respond(q);
    var translatedReply = translateText(reply, translatorLangState[0]);
    messagesState[1](function(prev) {
      return prev.concat([
        { id: 'u' + Date.now(), from: 'user', text: q },
        { id: 'a' + Date.now(), from: 'ai', text: translatedReply }
      ]);
    });
    inputState[1]('');
    var langMap = { 'en': 'en-US', 'hi': 'hi-IN', 'kn': 'kn-IN', 'ta': 'ta-IN', 'te': 'te-IN', 'ml': 'ml-IN', 'mr': 'mr-IN', 'gu': 'gu-IN', 'bn': 'bn-IN', 'pa': 'pa-IN' };
    speechHook.speak(translatedReply, { language: langMap[translatorLangState[0]] || 'en-US' }).catch(function() {});
  };

  var filteredLanguages = searchState[0]
    ? LANGUAGES.filter(function(l) { return l.nativeName.toLowerCase().indexOf(searchState[0].toLowerCase()) > -1 || l.label.toLowerCase().indexOf(searchState[0].toLowerCase()) > -1; })
    : LANGUAGES;

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Kisan AI Assistant'),
      React.createElement(TouchableOpacity, {
        onPress: function() { showTranslatorState[1](!showTranslatorState[0]); },
        style: { paddingHorizontal: 8, paddingVertical: 4, borderRadius: 8, backgroundColor: showTranslatorState[0] ? ACCENT_COLOR : 'rgba(255,255,255,0.15)' },
        componentId: 'translator-toggle'
      },
        React.createElement(Ionicons, { name: 'language', size: 18, color: showTranslatorState[0] ? '#1F2937' : '#FFFFFF' })
      )
    ),
    showTranslatorState[0] ? React.createElement(View, { style: { backgroundColor: '#F5F1E6', paddingHorizontal: 16, paddingVertical: 12, borderBottomWidth: 1, borderBottomColor: '#E5DCC8' } },
      React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: '#6B4E00', marginBottom: 8 } }, 'Translator & Language Selection'),
      React.createElement(TextInput, {
        style: [styles.input, { marginBottom: 8 }],
        value: searchState[0],
        onChangeText: searchState[1],
        placeholder: 'Search language...',
        componentId: 'language-search'
      }),
      React.createElement(ScrollView, { horizontal: true, showsHorizontalScrollIndicator: false, style: { marginBottom: 0 } },
        filteredLanguages.map(function(l) {
          var active = translatorLangState[0] === l.id;
          return React.createElement(TouchableOpacity, {
            key: l.id,
            onPress: function() { translatorLangState[1](l.id); },
            style: { paddingVertical: 6, paddingHorizontal: 12, borderRadius: 16, backgroundColor: active ? PRIMARY_COLOR : '#FFFFFF', marginRight: 8, borderWidth: 1, borderColor: active ? PRIMARY_COLOR : '#D1D5DB' },
            componentId: 'lang-chip-' + l.id
          },
            React.createElement(Text, { style: { color: active ? '#FFFFFF' : '#1F2937', fontSize: 11, fontWeight: '700' } }, l.label)
          );
        })
      )
    ) : null,
    React.createElement(ScrollView, { style: { flex: 1 }, contentContainerStyle: { padding: 16, paddingBottom: 16 } },
      messagesState[0].map(function(m) {
        return React.createElement(View, {
          key: m.id,
          style: { alignSelf: m.from === 'ai' ? 'flex-start' : 'flex-end', backgroundColor: m.from === 'ai' ? theme.colors.card : PRIMARY_COLOR, borderRadius: 14, padding: 12, marginBottom: 8, maxWidth: '85%' }
        },
          React.createElement(Text, { style: { color: m.from === 'ai' ? theme.colors.textPrimary : '#FFFFFF', fontSize: 13 } }, m.text)
        );
      })
    ),
    React.createElement(View, { style: { flexDirection: 'row', padding: 12, paddingBottom: insets.bottom + 12, borderTopWidth: 1, borderTopColor: '#EEE', backgroundColor: theme.colors.card } },
      React.createElement(TextInput, {
        style: [styles.input, { flex: 1, marginRight: 8 }],
        value: inputState[0],
        onChangeText: inputState[1],
        placeholder: 'Ask Kisan AI...',
        componentId: 'ai-chat-input'
      }),
      React.createElement(TouchableOpacity, {
        onPress: function() {
          if (!speechHook.isSttAvailable) { sendMessage(); return; }
          speechHook.startListening({ lang: 'en-US' }).catch(function() {});
        },
        style: { width: 44, height: 44, borderRadius: 22, backgroundColor: speechHook.isListening ? ACCENT_COLOR : '#EEF2E9', alignItems: 'center', justifyContent: 'center', marginRight: 8 },
        componentId: 'ai-mic-btn'
      },
        React.createElement(Ionicons, { name: 'mic', size: 20, color: speechHook.isListening ? '#1F2937' : PRIMARY_COLOR })
      ),
      React.createElement(TouchableOpacity, {
        onPress: sendMessage,
        style: { width: 44, height: 44, borderRadius: 22, backgroundColor: PRIMARY_COLOR, alignItems: 'center', justifyContent: 'center' },
        componentId: 'ai-send-btn'
      },
        React.createElement(Ionicons, { name: 'send', size: 18, color: '#FFFFFF' })
      )
    )
  );
};
// @end:EnhancedKisanAIScreen

// @section:KisanAIScreen @depends:[ThemeContext,styles]
var KisanAIScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var appointmentStorage = useStorage('farmerAppointment', DEFAULT_APPOINTMENT);
  var appointment = appointmentStorage[0];
  var langState = useState('en');
  var inputState = useState('');
  var messagesState = useState([
    { id: 'm0', from: 'ai', text: 'Namaste! Ask me about your token, queue, or waiting time.' }
  ]);
  var speechHook = useSpeech();

  var respond = function(question) {
    var lower = question.toLowerCase();
    var text;
    if (!appointment) {
      text = 'You have no active appointment yet. Please book a procurement slot first.';
    } else if (lower.indexOf('token') > -1 || lower.indexOf('टोकन') > -1 || lower.indexOf('ಟೋಕನ್') > -1) {
      text = 'Your token is ' + appointment.token + '.';
    } else if (lower.indexOf('wait') > -1 || lower.indexOf('turn') > -1 || lower.indexOf('सरदी') > -1 || lower.indexOf('ಸರದಿ') > -1 || lower.indexOf('बारी') > -1) {
      text = 'Your token is ' + appointment.token + '. Estimated waiting time is about ' + appointment.waitMinutes + ' minutes, queue position ' + appointment.queuePosition + '.';
    } else if (lower.indexOf('centre') > -1 || lower.indexOf('center') > -1) {
      text = 'Your procurement centre is ' + appointment.centreName + '.';
    } else {
      text = 'Your current status: ' + (STATUS_LABELS[appointment.status] || appointment.status) + ' at ' + appointment.centreName + '.';
    }
    if (langState[0] === 'kn') {
      text = 'ನಿಮ್ಮ ಟೋಕನ್ ' + (appointment ? appointment.token : '') + '. ' + (appointment ? ('ಸರದಿ ಬರಲು ಅಂದಾಜು ' + appointment.waitMinutes + ' ನಿಮಿಷಗಳಿವೆ.') : '');
    } else if (langState[0] === 'hi') {
      text = appointment ? ('आपका टोकन ' + appointment.token + ' है। अनुमानित प्रतीक्षा समय लगभग ' + appointment.waitMinutes + ' मिनट है।') : 'कोई सक्रिय अपॉइंटमेंट नहीं है।';
    }
    return text;
  };

  var sendMessage = function() {
    var q = inputState[0].trim();
    if (!q) return;
    var reply = respond(q);
    messagesState[1](function(prev) { return prev.concat([{ id: 'u' + Date.now(), from: 'user', text: q }, { id: 'a' + Date.now(), from: 'ai', text: reply }]); });
    inputState[1]('');
    speechHook.speak(reply, { language: langState[0] === 'kn' ? 'kn-IN' : (langState[0] === 'hi' ? 'hi-IN' : 'en-US') }).catch(function() {});
  };

  var LANGS = [{ id: 'en', label: 'EN' }, { id: 'hi', label: 'हिं' }, { id: 'kn', label: 'ಕನ್ನಡ' }];
  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Kisan AI'),
      React.createElement(View, { style: { flexDirection: 'row' } },
        LANGS.map(function(l) {
          var active = langState[0] === l.id;
          return React.createElement(TouchableOpacity, { key: l.id, onPress: function() { langState[1](l.id); }, style: { paddingHorizontal: 8, paddingVertical: 4, borderRadius: 8, backgroundColor: active ? ACCENT_COLOR : 'rgba(255,255,255,0.15)', marginLeft: 4 }, componentId: 'lang-' + l.id },
            React.createElement(Text, { style: { color: active ? '#1F2937' : '#FFFFFF', fontSize: 11, fontWeight: '700' } }, l.label)
          );
        })
      )
    ),
    React.createElement(ScrollView, { style: { flex: 1 }, contentContainerStyle: { padding: 16, paddingBottom: 16 } },
      messagesState[0].map(function(m) {
        return React.createElement(View, {
          key: m.id,
          style: { alignSelf: m.from === 'ai' ? 'flex-start' : 'flex-end', backgroundColor: m.from === 'ai' ? theme.colors.card : PRIMARY_COLOR, borderRadius: 14, padding: 12, marginBottom: 8, maxWidth: '85%' }
        }, React.createElement(Text, { style: { color: m.from === 'ai' ? theme.colors.textPrimary : '#FFFFFF', fontSize: 13 } }, m.text));
      })
    ),
    React.createElement(View, { style: { flexDirection: 'row', padding: 12, paddingBottom: insets.bottom + 12, borderTopWidth: 1, borderTopColor: '#EEE', backgroundColor: theme.colors.card } },
      React.createElement(TextInput, { style: [styles.input, { flex: 1, marginRight: 8 }], value: inputState[0], onChangeText: inputState[1], placeholder: 'Ask Kisan AI...', componentId: 'ai-chat-input' }),
      React.createElement(TouchableOpacity, {
        onPress: function() {
          if (!speechHook.isSttAvailable) { sendMessage(); return; }
          speechHook.startListening({ lang: 'en-US' }).catch(function() {});
        },
        style: { width: 44, height: 44, borderRadius: 22, backgroundColor: speechHook.isListening ? ACCENT_COLOR : '#EEF2E9', alignItems: 'center', justifyContent: 'center', marginRight: 8 },
        componentId: 'ai-mic-btn'
      }, React.createElement(Ionicons, { name: 'mic', size: 20, color: speechHook.isListening ? '#1F2937' : PRIMARY_COLOR })),
      React.createElement(TouchableOpacity, { onPress: sendMessage, style: { width: 44, height: 44, borderRadius: 22, backgroundColor: PRIMARY_COLOR, alignItems: 'center', justifyContent: 'center' }, componentId: 'ai-send-btn' },
        React.createElement(Ionicons, { name: 'send', size: 18, color: '#FFFFFF' })
      )
    )
  );
};
// @end:KisanAIScreen

// @section:FarmerTabNavigator @depends:[FarmerDashboardScreen,BookAppointmentScreen,TrackTokenScreen,PaymentScreen,NotificationsScreen,KisanAIScreen]
var FarmerTabNavigator = function() {
  var insets = useSafeAreaInsets();
  return React.createElement(Tab.Navigator, {
    screenOptions: function(routeProps) {
      var routeName = routeProps.route.name;
      return {
        headerShown: false,
        tabBarItemStyle: { padding: 0 },
        tabBarActiveTintColor: PRIMARY_COLOR,
        tabBarInactiveTintColor: TEXT_SECONDARY,
        tabBarStyle: { position: 'absolute', bottom: 0, height: Platform.OS === 'web' ? TAB_MENU_HEIGHT : TAB_MENU_HEIGHT + insets.bottom, paddingBottom: 0, borderTopWidth: 0, backgroundColor: '#FFFFFF' },
        tabBarIcon: function(iconProps) {
          var iconMap = { Dashboard: 'home', Book: 'add-circle', Track: 'locate', Payments: 'card', Notifications: 'notifications', Assistant: 'chatbubble-ellipses' };
          return React.createElement(Ionicons, { name: iconMap[routeName] || 'ellipse', size: 22, color: iconProps.color });
        }
      };
    }
  },
    React.createElement(Tab.Screen, { name: 'Dashboard', component: FarmerDashboardScreen }),
    React.createElement(Tab.Screen, { name: 'Book', component: BookAppointmentScreen }),
    React.createElement(Tab.Screen, { name: 'Track', component: TrackTokenScreen }),
    React.createElement(Tab.Screen, { name: 'Payments', component: PaymentScreen }),
    React.createElement(Tab.Screen, { name: 'Notifications', component: NotificationsScreen }),
    React.createElement(Tab.Screen, { name: 'Assistant', component: KisanAIScreen })
  );
};
// @end:FarmerTabNavigator

// @section:TransportOptimizationScreen @depends:[ThemeContext,constants,styles]
var TransportOptimizationScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var expandedState = useState(null);
  var expanded = expandedState[0];
  var setExpanded = expandedState[1];

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  var routes = [
    { id: 'r1', from: 'Centre A', to: 'Warehouse B', quantity: 150, vehicle: 'Truck (20MT)', distance: 45, eta: '14:30', status: 'in_transit' },
    { id: 'r2', from: 'Centre B', to: 'Warehouse A', quantity: 75, vehicle: 'Truck (10MT)', distance: 32, eta: '15:45', status: 'scheduled' },
    { id: 'r3', from: 'Centre C', to: 'Warehouse C', quantity: 120, vehicle: 'Truck (20MT)', distance: 28, eta: '13:15', status: 'completed' }
  ];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Smart Transport Optimization')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap', marginHorizontal: -6, marginBottom: 12 } },
        [
          { label: 'Active Routes', value: '3' },
          { label: 'Total Capacity', value: '450 MT' },
          { label: 'Avg Utilization', value: '82%' }
        ].map(function(k, idx) {
          return React.createElement(View, { key: String(idx), style: { width: '33.33%', padding: 6 } },
            React.createElement(View, { style: [styles.card, { padding: 12, alignItems: 'center' }] },
              React.createElement(Text, { style: { fontSize: 10, color: theme.colors.textSecondary } }, k.label),
              React.createElement(Text, { style: { fontSize: 16, fontWeight: '900', color: theme.colors.textPrimary, marginTop: 4 } }, k.value)
            )
          );
        })
      ),
      React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 8 } }, 'Active Routes'),
      routes.map(function(r) {
        var isExpanded = expanded === r.id;
        var statusColor = r.status === 'completed' ? '#16A34A' : (r.status === 'in_transit' ? ACCENT_COLOR : '#6B7280');
        return React.createElement(TouchableOpacity, {
          key: r.id,
          onPress: function() { setExpanded(isExpanded ? null : r.id); },
          style: [styles.card, { marginBottom: 10 }],
          componentId: 'route-' + r.id
        },
          React.createElement(View, { style: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
            React.createElement(View, { style: { flex: 1 } },
              React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary } }, r.from + ' → ' + r.to),
              React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, r.quantity + ' MT · ' + r.distance + ' km')
            ),
            React.createElement(View, { style: { alignItems: 'flex-end' } },
              React.createElement(Text, { style: { fontSize: 12, fontWeight: '800', color: statusColor } }, r.status.replace('_', ' ').toUpperCase()),
              React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, 'ETA ' + r.eta)
            )
          ),
          isExpanded ? React.createElement(View, { style: { marginTop: 12, paddingTop: 12, borderTopWidth: 1, borderTopColor: '#E5E7EB' } },
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary } }, 'Vehicle: ' + r.vehicle),
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, 'Utilization: ' + Math.round((r.quantity / 20) * 100) + '%'),
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, 'Efficiency Score: 94%')
          ) : null
        );
      }),
      React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary, marginTop: 12, marginBottom: 8 } }, 'Available Vehicles'),
      VEHICLES.map(function(v) {
        return React.createElement(View, { key: v.id, style: [styles.card, { marginBottom: 8, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' }] },
          React.createElement(View, null,
            React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary } }, v.type),
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, 'Capacity: ' + v.capacity + ' MT')
          ),
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: PRIMARY_COLOR } }, v.available + ' available')
        );
      })
    )
  );
};
// @end:TransportOptimizationScreen

// @section:WarehouseManagementScreen @depends:[ThemeContext,constants,styles]
var WarehouseManagementScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Warehouse Capacity Prediction')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: [styles.card, { marginBottom: 12, borderLeftWidth: 4, borderLeftColor: ACCENT_COLOR }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: '#92600A' } }, '🔮 AI PREDICTION — 7 DAYS'),
        React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 6 } }, 'Total network capacity trending toward 78% utilization by Day 7.')
      ),
      WAREHOUSES.map(function(wh) {
        var utilization = (wh.current / wh.capacity) * 100;
        var predicted = utilization + Math.random() * 15;
        var color = predicted >= 85 ? '#DC2626' : (predicted >= 60 ? ACCENT_COLOR : '#16A34A');
        return React.createElement(View, { key: wh.id, style: [styles.card, { marginBottom: 12 }] },
          React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary } }, wh.name),
          React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, wh.location + ' · ' + wh.type),
          React.createElement(View, { style: { marginTop: 8 } },
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginBottom: 4 } }, 'Current: ' + Math.round(utilization) + '% · Predicted: ' + Math.round(predicted) + '%'),
            React.createElement(View, { style: { height: 8, borderRadius: 4, backgroundColor: '#EEE', overflow: 'hidden' } },
              React.createElement(View, { style: { width: Math.round(utilization) + '%', height: 8, backgroundColor: loadColor(utilization) } })
            )
          ),
          predicted >= 85 ? React.createElement(View, { style: { backgroundColor: '#FEE2E2', borderRadius: 8, padding: 8, marginTop: 8 } },
            React.createElement(Text, { style: { fontSize: 11, color: '#991B1B', fontWeight: '600' } }, '⚠️ AI Recommendation: Redirect 180 MT to ' + (wh.id === 'wh-a' ? 'Warehouse B' : 'Warehouse A'))
          ) : null
        );
      }),
      React.createElement(View, { style: [styles.card, { marginTop: 12, backgroundColor: '#EEF2E9' }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: PRIMARY_COLOR } }, 'Network Optimization Opportunity'),
        React.createElement(Text, { style: { fontSize: 12, color: PRIMARY_COLOR, marginTop: 6 } }, 'Redistribute 180 MT of paddy from Centre A overflow to Warehouse B to balance network capacity and reduce spoilage risk by 18%.')
      )
    )
  );
};
// @end:WarehouseManagementScreen

// @section:EmergencyProcurementScreen @depends:[ThemeContext,constants,styles]
var EmergencyProcurementScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var emergencyActiveState = useState(false);
  var emergencyActive = emergencyActiveState[0];
  var setEmergencyActive = emergencyActiveState[1];
  var emergencyTypeState = useState('none');

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  var EMERGENCY_TYPES = [
    { id: 'harvest-surge', label: 'Harvest Surge', desc: 'Sudden influx of farmers & crops' },
    { id: 'centre-closure', label: 'Centre Closure', desc: 'One or more centres unavailable' },
    { id: 'transport-disruption', label: 'Transport Disruption', desc: 'Vehicle shortage or road damage' },
    { id: 'severe-weather', label: 'Severe Weather', desc: 'Flooding, extreme heat, or storms' }
  ];

  var activateEmergency = function(type) {
    emergencyTypeState[1](type);
    setEmergencyActive(true);
  };

  var deactivateEmergency = function() {
    setEmergencyActive(false);
    emergencyTypeState[1]('none');
  };

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Emergency Procurement Mode')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      emergencyActive ? React.createElement(View, null,
        React.createElement(View, { style: { backgroundColor: '#FEE2E2', borderLeftWidth: 4, borderLeftColor: '#DC2626', borderRadius: 12, padding: 14, marginBottom: 14 } },
          React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: '#991B1B' } }, '🚨 EMERGENCY MODE ACTIVE'),
          React.createElement(Text, { style: { fontSize: 12, color: '#991B1B', marginTop: 6 } }, 'Scenario: ' + EMERGENCY_TYPES.filter(function(e) { return e.id === emergencyTypeState[0]; })[0].label)
        ),
        React.createElement(View, { style: [styles.card, { marginBottom: 12 }] },
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 8 } }, 'AI AUTOMATED RESPONSE'),
          [
            { label: 'Disruption Detected', desc: 'Centre A capacity exceeded 120%' },
            { label: 'Capacity Calculated', desc: '650 farmers waiting, 120 MT queued' },
            { label: 'Redistribution Initiated', desc: 'Move 200 farmers to Centre B/C' },
            { label: 'Transport Reassigned', desc: '5 additional vehicles deployed' },
            { label: 'Notifications Sent', desc: '650 farmers updated with new slots' },
            { label: 'Recovery Monitoring', desc: 'Centre A load reduced to 82%' }
          ].map(function(item, idx) {
            return React.createElement(View, { key: String(idx), style: { flexDirection: 'row', marginBottom: 8, paddingBottom: 8, borderBottomWidth: idx === 5 ? 0 : 1, borderBottomColor: '#E5E7EB' } },
              React.createElement(View, { style: { width: 24, height: 24, borderRadius: 12, backgroundColor: PRIMARY_COLOR, alignItems: 'center', justifyContent: 'center', marginRight: 10 } },
                React.createElement(Ionicons, { name: 'checkmark', size: 14, color: '#FFFFFF' })
              ),
              React.createElement(View, null,
                React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: theme.colors.textPrimary } }, item.label),
                React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, item.desc)
              )
            );
          })
        ),
        React.createElement(TouchableOpacity, {
          style: { backgroundColor: '#DC2626', borderRadius: 10, padding: 14, alignItems: 'center' },
          onPress: deactivateEmergency,
          componentId: 'deactivate-emergency'
        },
          React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '800' } }, 'DEACTIVATE EMERGENCY MODE')
        )
      ) : React.createElement(View, null,
        React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 8 } }, 'Emergency Scenarios'),
        EMERGENCY_TYPES.map(function(e) {
          return React.createElement(TouchableOpacity, {
            key: e.id,
            onPress: function() { activateEmergency(e.id); },
            style: [styles.card, { marginBottom: 10, flexDirection: 'row', alignItems: 'center' }],
            componentId: 'emergency-' + e.id
          },
            React.createElement(View, { style: { width: 48, height: 48, borderRadius: 24, backgroundColor: '#FEE2E2', alignItems: 'center', justifyContent: 'center', marginRight: 12 } },
              React.createElement(Text, { style: { fontSize: 24 } }, e.id === 'harvest-surge' ? '📈' : (e.id === 'centre-closure' ? '🔴' : (e.id === 'transport-disruption' ? '🚗' : '⛈️')))
            ),
            React.createElement(View, { style: { flex: 1 } },
              React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary } }, e.label),
              React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, e.desc)
            ),
            React.createElement(Ionicons, { name: 'chevron-forward', size: 18, color: theme.colors.textSecondary })
          );
        })
      )
    )
  );
};
// @end:EmergencyProcurementScreen

// @section:PolicySimulatorScreen @depends:[ThemeContext,constants,styles]
var PolicySimulatorScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  
  var demandChangeState = useState('0');
  var closedCentresState = useState('0');
  var staffChangeState = useState('0');
  var resultState = useState(null);
  var runningState = useState(false);

  var runPolicySimulation = function() {
    runningState[1](true);
    setTimeout(function() {
      var demandChange = parseInt(demandChangeState[0], 10) || 0;
      var closedCentres = parseInt(closedCentresState[0], 10) || 0;
      var staffChange = parseInt(staffChangeState[0], 10) || 0;

      var baseWait = 34;
      var predictedWait = Math.round(baseWait * (1 + demandChange / 100) * (1 + closedCentres * 0.15) * (1 + staffChange / 100));
      var centreA = 71 + demandChange / 2 - staffChange / 2;
      var transportReq = 100 + demandChange * 0.18;
      var warehouseUtil = 82 + demandChange * 0.12;
      
      resultState[1]({
        currentWait: baseWait,
        predictedWait: predictedWait,
        centreALoad: Math.round(centreA),
        transportReq: Math.round(transportReq),
        warehouseUtil: Math.round(warehouseUtil),
        recommendation: demandChange > 20 || closedCentres > 0 ? 'Open temporary centre or increase staff by ' + Math.ceil((demandChange + closedCentres * 30) / 10) * 10 + '%' : 'Maintain current operations'
      });
      runningState[1](false);
    }, 1200);
  };

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Government Policy Simulator')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textSecondary, marginBottom: 8 } }, 'Adjust policy parameters to simulate impact'),
      React.createElement(Text, { style: styles.label }, 'Demand Change (%)'),
      React.createElement(TextInput, { 
        style: styles.input, 
        value: demandChangeState[0], 
        onChangeText: function(t) { demandChangeState[1](t.replace(/[^0-9\-]/g, '')); }, 
        placeholder: '0', 
        keyboardType: 'numeric',
        componentId: 'sim-demand'
      }),
      React.createElement(Text, { style: styles.label }, 'Unavailable Centres'),
      React.createElement(TextInput, { 
        style: styles.input, 
        value: closedCentresState[0], 
        onChangeText: function(t) { closedCentresState[1](t.replace(/[^0-9]/g, '')); }, 
        placeholder: '0', 
        keyboardType: 'numeric',
        componentId: 'sim-closed'
      }),
      React.createElement(Text, { style: styles.label }, 'Staff Change (%)'),
      React.createElement(TextInput, { 
        style: styles.input, 
        value: staffChangeState[0], 
        onChangeText: function(t) { staffChangeState[1](t.replace(/[^0-9\-]/g, '')); }, 
        placeholder: '0', 
        keyboardType: 'numeric',
        componentId: 'sim-staff'
      }),
      React.createElement(TouchableOpacity, { 
        style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 14, alignItems: 'center', marginTop: 12 }, 
        onPress: runPolicySimulation,
        componentId: 'run-policy-sim'
      },
        runningState[0] ? React.createElement(ActivityIndicator, { color: '#1F2937' }) : React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800' } }, 'RUN SIMULATION')
      ),
      resultState[0] ? React.createElement(View, { style: [styles.card, { marginTop: 14 }] },
        React.createElement(View, { style: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 10, paddingBottom: 10, borderBottomWidth: 1, borderBottomColor: '#E5E7EB' } },
          React.createElement(View, null,
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'Current Wait'),
            React.createElement(Text, { style: { fontSize: 18, fontWeight: '900', color: theme.colors.textPrimary } }, resultState[0].currentWait + ' min')
          ),
          React.createElement(View, null,
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'Predicted Wait'),
            React.createElement(Text, { style: { fontSize: 18, fontWeight: '900', color: loadColor(resultState[0].predictedWait > 70 ? 90 : 50) } }, resultState[0].predictedWait + ' min')
          )
        ),
        [
          { label: 'Centre A Load', value: resultState[0].centreALoad + '%' },
          { label: 'Transport Demand', value: resultState[0].transportReq + '%' },
          { label: 'Warehouse Util.', value: resultState[0].warehouseUtil + '%' }
        ].map(function(item, idx) {
          return React.createElement(View, { key: String(idx), style: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 6 } },
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary } }, item.label),
            React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: theme.colors.textPrimary } }, item.value)
          );
        }),
        React.createElement(View, { style: { backgroundColor: '#EEF2E9', borderRadius: 8, padding: 10, marginTop: 10 } },
          React.createElement(Text, { style: { fontSize: 12, color: '#33501B', fontWeight: '600' } }, 'AI Recommendation'),
          React.createElement(Text, { style: { fontSize: 12, color: '#33501B', marginTop: 4 } }, resultState[0].recommendation)
        )
      ) : null
    )
  );
};
// @end:PolicySimulatorScreen

// @section:OfficerAssistantScreen @depends:[ThemeContext,styles,constants]
var OfficerAssistantScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreState = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreState[0];
  
  var inputState = useState('');
  var messagesState = useState([
    { id: 'm0', from: 'ai', text: 'Welcome, Officer. I can help you with procurement insights, centre performance, queue predictions, and policy analysis. What would you like to know?' }
  ]);
  var speechHook = useSpeech();

  var respond = function(question) {
    var lower = question.toLowerCase();
    if (lower.indexOf('overloaded') > -1 || lower.indexOf('congestion') > -1) {
      return 'Centre A is predicted to reach 94% capacity tomorrow with a 91% probability of exceeding recommended limits. I recommend moving 85 appointments to Centre B to reduce congestion.';
    } else if (lower.indexOf('worst') > -1 || lower.indexOf('perform') > -1) {
      return 'Centre A has the highest utilization this week at 87% average. Centre B shows 43% utilization with available capacity. This week, 12,876 farmers were processed across all centres.';
    } else if (lower.indexOf('capacity') > -1 || lower.indexOf('warehouse') > -1) {
      return 'Warehouse A is at 72% capacity with 7-day prediction of 94%. Warehouse B is at 45% with 7-day prediction of 61%. I recommend redirecting 180 MT from Centre A overflow to Warehouse B.';
    } else if (lower.indexOf('transport') > -1 || lower.indexOf('route') > -1) {
      return 'Currently 3 active transport routes with 82% average vehicle utilization. Centre A to Warehouse B route shows optimal efficiency at 94%. All routes are operating within schedule.';
    } else if (lower.indexOf('emergency') > -1) {
      return 'Emergency mode can be activated for harvest surges, centre closures, transport disruptions, or severe weather. Once activated, the system automatically detects disruptions, recalculates capacity, redistributes appointments, and monitors recovery.';
    } else {
      return 'I can provide insights on centre load, capacity predictions, transport optimization, warehouse utilization, emergency protocols, and policy simulations. Try asking about congestion, performance, capacity, or emergency procedures.';
    }
  };

  var sendMessage = function() {
    var q = inputState[0].trim();
    if (!q) return;
    var reply = respond(q);
    messagesState[1](function(prev) {
      return prev.concat([
        { id: 'u' + Date.now(), from: 'user', text: q },
        { id: 'a' + Date.now(), from: 'ai', text: reply }
      ]);
    });
    inputState[1]('');
    speechHook.speak(reply, { language: 'en-US' }).catch(function() {});
  };

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Officer AI Assistant')
    ),
    React.createElement(ScrollView, { style: { flex: 1 }, contentContainerStyle: { padding: 16, paddingBottom: 16 } },
      messagesState[0].map(function(m) {
        return React.createElement(View, {
          key: m.id,
          style: { alignSelf: m.from === 'ai' ? 'flex-start' : 'flex-end', backgroundColor: m.from === 'ai' ? theme.colors.card : PRIMARY_COLOR, borderRadius: 14, padding: 12, marginBottom: 8, maxWidth: '85%' }
        },
          React.createElement(Text, { style: { color: m.from === 'ai' ? theme.colors.textPrimary : '#FFFFFF', fontSize: 13 } }, m.text)
        );
      })
    ),
    React.createElement(View, { style: { flexDirection: 'row', padding: 12, paddingBottom: insets.bottom + 12, borderTopWidth: 1, borderTopColor: '#EEE', backgroundColor: theme.colors.card } },
      React.createElement(TextInput, {
        style: [styles.input, { flex: 1, marginRight: 8 }],
        value: inputState[0],
        onChangeText: inputState[1],
        placeholder: 'Ask about operations...',
        componentId: 'officer-ai-input'
      }),
      React.createElement(TouchableOpacity, {
        onPress: function() {
          if (!speechHook.isSttAvailable) { sendMessage(); return; }
          speechHook.startListening({ lang: 'en-US' }).catch(function() {});
        },
        style: { width: 44, height: 44, borderRadius: 22, backgroundColor: speechHook.isListening ? ACCENT_COLOR : '#EEF2E9', alignItems: 'center', justifyContent: 'center', marginRight: 8 },
        componentId: 'officer-ai-mic'
      },
        React.createElement(Ionicons, { name: 'mic', size: 20, color: speechHook.isListening ? '#1F2937' : PRIMARY_COLOR })
      ),
      React.createElement(TouchableOpacity, {
        onPress: sendMessage,
        style: { width: 44, height: 44, borderRadius: 22, backgroundColor: PRIMARY_COLOR, alignItems: 'center', justifyContent: 'center' },
        componentId: 'officer-ai-send'
      },
        React.createElement(Ionicons, { name: 'send', size: 18, color: '#FFFFFF' })
      )
    )
  );
};
// @end:OfficerAssistantScreen

// @section:CommandCentreScreen @depends:[ThemeContext,constants,styles]
var CommandCentreScreen = function(props) {
  var navigation = props.navigation;
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreStorage[0];
  var roleState = useStorage('activeRole', null);
  var setRole = roleState[1];

  var totalFarmers = 18452;
  var completed = 12876;
  var currentWaiting = Object.keys(centreLoads).reduce(function(sum, k) { return sum + centreLoads[k].queue; }, 0);
  var avgWaiting = 31;

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  var handleSwitchRole = function() {
    setRole(null);
    // User stays logged in, just role is cleared - will show RoleSelectScreen
  };

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 17, fontWeight: '800' } }, 'SMART PROCUREMENT COMMAND CENTRE'),
      React.createElement(TouchableOpacity, { onPress: handleSwitchRole, componentId: 'officer-switch-role' },
        React.createElement(Ionicons, { name: 'swap-horizontal', size: 20, color: '#FFFFFF' })
      )
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap', marginHorizontal: -6 } },
        [
          { label: 'Total Centres', value: String(CENTRES.length) },
          { label: 'Farmers Today', value: '18,452' },
          { label: 'Completed', value: '12,876' },
          { label: 'Current Waiting', value: String(currentWaiting) },
          { label: 'Average Waiting', value: avgWaiting + ' min' }
        ].map(function(k, idx) {
          return React.createElement(View, { key: String(idx), style: { width: '50%', padding: 6 } },
            React.createElement(View, { style: [styles.card, { padding: 14 }] },
              React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, k.label),
              React.createElement(Text, { style: { fontSize: 20, fontWeight: '900', color: theme.colors.textPrimary, marginTop: 4 } }, k.value)
            )
          );
        })
      ),
      React.createElement(Text, { style: { fontSize: 15, fontWeight: '800', color: theme.colors.textPrimary, marginTop: 10, marginBottom: 8 } }, 'Centre Load Overview'),
      CENTRES.map(function(c) {
        var st = centreLoads[c.id];
        var color = loadColor(st.predicted);
        return React.createElement(TouchableOpacity, {
          key: c.id,
          onPress: function() { navigation.navigate('CentreDetail', { centreId: c.id }); },
          style: [styles.card, { marginBottom: 10, flexDirection: 'row', alignItems: 'center' }],
          componentId: 'command-centre-card-' + c.id
        },
          React.createElement(View, { style: { width: 12, height: 12, borderRadius: 6, backgroundColor: color, marginRight: 12 } }),
          React.createElement(View, { style: { flex: 1 } },
            React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary } }, c.name),
            React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 2 } }, 'Current ' + st.current + '% · Predicted ' + st.predicted + '% · Queue ' + st.queue)
          ),
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: color } }, loadLabel(st.predicted)),
          React.createElement(Ionicons, { name: 'chevron-forward', size: 18, color: theme.colors.textSecondary, style: { marginLeft: 8 } })
        );
      }),
      React.createElement(View, { style: [styles.card, { marginTop: 6, borderLeftWidth: 4, borderLeftColor: ACCENT_COLOR }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: '#92600A' } }, 'AI PREDICTIONS — TOMORROW'),
        React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 6 } }, 'Expected Farmers: 620 · Expected Procurement: 1,540 tonnes'),
        React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 2 } }, 'Peak: 10:00 AM – 1:00 PM · Peak Congestion: 91%')
      )
    )
  );
};
// @end:CommandCentreScreen

// @section:RecommendationsScreen-handlers @depends:[]
var recommendationsHandlers = {
  approve: function(rec, centreLoads, setCentreLoads, updateRecommendation, insertNotification, insertAudit, refetch) {
    var fromCentre = CENTRES[0];
    var toB = CENTRES[1];
    var toC = CENTRES[2];
    setCentreLoads(function(prev) {
      var next = Object.assign({}, prev);
      next[fromCentre.id] = Object.assign({}, prev[fromCentre.id], { current: Math.max(30, prev[fromCentre.id].current - 23), predicted: Math.max(35, prev[fromCentre.id].predicted - 23), queue: Math.max(20, prev[fromCentre.id].queue - 90) });
      next[toB.id] = Object.assign({}, prev[toB.id], { current: prev[toB.id].current + 22, predicted: prev[toB.id].predicted + 22, queue: prev[toB.id].queue + 75 });
      next[toC.id] = Object.assign({}, prev[toC.id], { current: prev[toC.id].current + 4, predicted: prev[toC.id].predicted + 4, queue: prev[toC.id].queue + 45 });
      return next;
    });
    updateRecommendation({ id: rec.id, data: { status: 'approved', approval_timestamp: new Date().toISOString(), execution_timestamp: new Date().toISOString() } })
      .then(function() {
        insertAudit({ action: 'recommendation_approved', entity_type: 'ai_recommendations', entity_id: rec.id, changes: { status: 'approved' } }).catch(function() {});
        insertNotification({ farmer_id: genId(), notification_type: 'appointment_rescheduled', title: 'Appointment Reassigned', message: 'Your appointment has been moved to reduce congestion. New expected waiting is approximately 31 minutes.', is_read: false }).catch(function() {});
        refetch();
      });
  },
  reject: function(rec, updateRecommendation, insertAudit, refetch) {
    updateRecommendation({ id: rec.id, data: { status: 'rejected' } }).then(function() {
      insertAudit({ action: 'recommendation_rejected', entity_type: 'ai_recommendations', entity_id: rec.id, changes: { status: 'rejected' } }).catch(function() {});
      refetch();
    });
  },
  generate: function(insertRecommendation, refetch) {
    insertRecommendation({
      recommendation_type: 'load_balance',
      description: 'Centre A is expected to reach 94% capacity tomorrow. Shift 85 appointments from Centre A to Centre B/C between 10:00–13:00.',
      affected_farmers_count: 85,
      expected_impact: { avgWaitBefore: 78, avgWaitAfter: 31, centreABefore: 94, centreAAfter: 71 },
      explanation: 'Centre A predicted load 94%, Centre C predicted load 43%, average distance 7.2km, expected waiting reduction 47 min.',
      status: 'pending'
    }).then(refetch);
  }
};
// @end:RecommendationsScreen-handlers

// @section:RecommendationsScreen @depends:[ThemeContext,RecommendationsScreen-handlers,styles]
var RecommendationsScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var { data: recommendations, loading, refetch } = useQuery('ai_recommendations', {}, { column: 'created_at', ascending: false });
  var { mutate: insertRecommendation } = useMutation('ai_recommendations', 'insert');
  var { mutate: updateRecommendation } = useMutation('ai_recommendations', 'update');
  var { mutate: insertNotification } = useMutation('notifications', 'insert');
  var { mutate: insertAudit } = useMutation('audit_logs', 'insert');
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var setCentreLoads = centreStorage[1];
  var centreLoads = centreStorage[0];

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var list = (recommendations && recommendations.length > 0) ? recommendations : [];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'AI Recommendations')
    ),
    loading ? React.createElement(ActivityIndicator, { style: { flex: 1 }, componentId: 'reco-loading' }) :
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(TouchableOpacity, {
        style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 14, alignItems: 'center', marginBottom: 14 },
        onPress: function() { recommendationsHandlers.generate(insertRecommendation, refetch); },
        componentId: 'generate-reco'
      }, React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800' } }, 'RUN CONGESTION ANALYSIS')),
      list.length === 0 ? React.createElement(Text, { style: { color: theme.colors.textSecondary, textAlign: 'center', marginTop: 20 } }, 'No recommendations yet — run congestion analysis') :
      list.map(function(rec) {
        var impact = rec.expected_impact || {};
        return React.createElement(View, { key: rec.id, style: [styles.card, { marginBottom: 12, borderLeftWidth: 4, borderLeftColor: rec.status === 'approved' ? '#16A34A' : (rec.status === 'rejected' ? '#DC2626' : ACCENT_COLOR) }] },
          React.createElement(Text, { style: { fontSize: 12, fontWeight: '800', color: '#DC2626' } }, '🔴 HIGH CONGESTION PREDICTED'),
          React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textPrimary, marginTop: 6, lineHeight: 19 } }, rec.description),
          impact.avgWaitBefore ? React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 8 } }, 'Avg waiting: ' + impact.avgWaitBefore + ' → ' + impact.avgWaitAfter + ' min · Centre A: ' + impact.centreABefore + '% → ' + impact.centreAAfter + '%') : null,
          React.createElement(View, { style: { backgroundColor: '#F5F1E6', borderRadius: 8, padding: 10, marginTop: 8 } },
            React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00', fontWeight: '700' } }, 'What influenced this decision?'),
            React.createElement(Text, { style: { fontSize: 11, color: '#6B4E00', marginTop: 4 } }, rec.explanation)
          ),
          rec.status === 'pending' ? React.createElement(View, { style: { flexDirection: 'row', marginTop: 12 } },
            React.createElement(TouchableOpacity, { style: { flex: 1, backgroundColor: '#16A34A', borderRadius: 8, padding: 12, alignItems: 'center', marginRight: 6 }, onPress: function() { recommendationsHandlers.approve(rec, centreLoads, setCentreLoads, updateRecommendation, insertNotification, insertAudit, refetch); }, componentId: 'approve-' + rec.id },
              React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '800', fontSize: 12 } }, 'APPROVE')
            ),
            React.createElement(TouchableOpacity, { style: { flex: 1, backgroundColor: '#DC2626', borderRadius: 8, padding: 12, alignItems: 'center', marginLeft: 6 }, onPress: function() { recommendationsHandlers.reject(rec, updateRecommendation, insertAudit, refetch); }, componentId: 'reject-' + rec.id },
              React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '800', fontSize: 12 } }, 'REJECT')
            )
          ) : React.createElement(Text, { style: { fontSize: 12, fontWeight: '800', color: theme.colors.textSecondary, marginTop: 10 } }, 'Status: ' + rec.status.toUpperCase())
        );
      })
    )
  );
};
// @end:RecommendationsScreen

// @section:LoadBalancerScreen @depends:[ThemeContext,constants,styles]
var LoadBalancerScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreStorage[0];
  var whyState = useState(null);

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'AI Load Balancer')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      CENTRES.map(function(c) {
        var st = centreLoads[c.id];
        return React.createElement(View, { key: c.id, style: [styles.card, { marginBottom: 10 }] },
          React.createElement(View, { style: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
            React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary } }, c.name.replace('Procurement ', '')),
            React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: loadColor(st.current) } }, st.current + '% ' + (st.current >= 85 ? '🔴' : (st.current >= 60 ? '🟡' : '🟢')))
          ),
          React.createElement(View, { style: { height: 8, borderRadius: 4, backgroundColor: '#EEE', marginTop: 8 } },
            React.createElement(View, { style: { width: Math.min(100, st.current) + '%', height: 8, borderRadius: 4, backgroundColor: loadColor(st.current) } })
          )
        );
      }),
      React.createElement(View, { style: [styles.card, { marginTop: 6 }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 8 } }, 'Proposed Redistribution'),
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textPrimary } }, 'Move 75 farmers: Centre A → Centre B'),
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textPrimary, marginTop: 4 } }, 'Move 45 farmers: Centre A → Centre C'),
        React.createElement(TouchableOpacity, { onPress: function() { whyState[1](whyState[0] ? null : 'Distance and crop acceptance favor Centre B/C. Centre A predicted load 94% vs Centre B 51%, Centre C 68%. Expected waiting reduction ~47 minutes.'); }, style: { marginTop: 10 }, componentId: 'why-btn' },
          React.createElement(Text, { style: { color: PRIMARY_COLOR, fontWeight: '700', fontSize: 12 } }, whyState[0] ? 'Hide explanation' : 'Why? ')
        ),
        whyState[0] ? React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 6, lineHeight: 18 } }, whyState[0]) : null
      )
    )
  );
};
// @end:LoadBalancerScreen

// @section:SimulatorScreen @depends:[ThemeContext,styles]
var SimulatorScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var farmersState = useState('500');
  var countersState = useState('4');
  var staffState = useState('20');
  var resultState = useState(null);
  var runningState = useState(false);

  var runSimulation = function() {
    runningState[1](true);
    setTimeout(function() {
      var farmers = parseInt(farmersState[0], 10) || 500;
      var counters = parseInt(countersState[0], 10) || 4;
      var staff = parseInt(staffState[0], 10) || 20;
      var baseWait = 32;
      var predictedWait = Math.round(baseWait * (farmers / 500) * (4 / counters) * (20 / Math.max(5, staff)));
      var congestion = predictedWait > 70 ? 'HIGH' : (predictedWait > 40 ? 'MODERATE' : 'LOW');
      var requiredCounters = Math.ceil(counters * (predictedWait / baseWait));
      resultState[1]({ currentWait: baseWait, predictedWait: predictedWait, congestion: congestion, requiredCounters: requiredCounters });
      runningState[1](false);
    }, 1200);
  };

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'AI What-If Simulator')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(Text, { style: styles.label }, 'Expected Farmers'),
      React.createElement(TextInput, { style: styles.input, value: farmersState[0], onChangeText: function(t) { farmersState[1](t.replace(/[^0-9]/g, '')); }, keyboardType: 'numeric', componentId: 'sim-farmers' }),
      React.createElement(Text, { style: styles.label }, 'Available Counters'),
      React.createElement(TextInput, { style: styles.input, value: countersState[0], onChangeText: function(t) { countersState[1](t.replace(/[^0-9]/g, '')); }, keyboardType: 'numeric', componentId: 'sim-counters' }),
      React.createElement(Text, { style: styles.label }, 'Staff'),
      React.createElement(TextInput, { style: styles.input, value: staffState[0], onChangeText: function(t) { staffState[1](t.replace(/[^0-9]/g, '')); }, keyboardType: 'numeric', componentId: 'sim-staff' }),
      React.createElement(TouchableOpacity, { style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 14, alignItems: 'center', marginTop: 8 }, onPress: runSimulation, componentId: 'run-simulation' },
        runningState[0] ? React.createElement(ActivityIndicator, { color: '#1F2937' }) : React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800' } }, 'RUN SIMULATION')
      ),
      resultState[0] ? React.createElement(View, { style: [styles.card, { marginTop: 14 }] },
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary } }, 'Current Waiting: ' + resultState[0].currentWait + ' minutes'),
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary, marginTop: 4 } }, 'Predicted Waiting: ' + resultState[0].predictedWait + ' minutes'),
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: loadColor(resultState[0].congestion === 'HIGH' ? 90 : (resultState[0].congestion === 'MODERATE' ? 65 : 30)), marginTop: 6 } }, 'Congestion: ' + resultState[0].congestion),
        React.createElement(Text, { style: { fontSize: 13, color: theme.colors.textSecondary, marginTop: 4 } }, 'Required Counters: ' + resultState[0].requiredCounters),
        React.createElement(View, { style: { backgroundColor: '#F5F1E6', borderRadius: 8, padding: 10, marginTop: 10 } },
          React.createElement(Text, { style: { fontSize: 12, color: '#6B4E00' } }, 'AI Recommendation: Add ' + Math.max(1, resultState[0].requiredCounters - (parseInt(countersState[0], 10) || 4)) + ' counters OR redistribute appointments.')
        )
      ) : null
    )
  );
};
// @end:SimulatorScreen

// @section:AnomaliesScreen @depends:[ThemeContext,styles]
var AnomaliesScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var { data: anomalies, loading, refetch } = useQuery('anomalies', {}, { column: 'created_at', ascending: false });
  var { mutate: insertAnomaly } = useMutation('anomalies', 'insert');
  var { mutate: updateAnomaly } = useMutation('anomalies', 'update');

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var list = (anomalies && anomalies.length > 0) ? anomalies : [];
  var severityColor = { low: '#16A34A', medium: '#F59E0B', high: '#DC2626' };

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'AI Anomaly Monitor')
    ),
    loading ? React.createElement(ActivityIndicator, { style: { flex: 1 }, componentId: 'anomaly-loading' }) :
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(TouchableOpacity, {
        style: { backgroundColor: ACCENT_COLOR, borderRadius: 10, padding: 14, alignItems: 'center', marginBottom: 14 },
        onPress: function() {
          insertAnomaly({
            anomaly_type: 'unusual_quantity', severity: 'high',
            description: 'Anomalous activity detected — requires official investigation.',
            expected_value: '2.5 tonnes', observed_value: '25 tonnes', status: 'detected'
          }).then(refetch);
        },
        componentId: 'simulate-anomaly'
      }, React.createElement(Text, { style: { color: '#1F2937', fontWeight: '800' } }, 'SIMULATE ANOMALY DETECTION')),
      list.length === 0 ? React.createElement(Text, { style: { color: theme.colors.textSecondary, textAlign: 'center', marginTop: 20 } }, 'No anomalies detected') :
      list.map(function(a) {
        return React.createElement(View, { key: a.id, style: [styles.card, { marginBottom: 10, borderLeftWidth: 4, borderLeftColor: severityColor[a.severity] || '#F59E0B' }] },
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: theme.colors.textPrimary } }, '⚠️ ' + a.description),
          React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 6 } }, 'Expected: ' + a.expected_value + ' · Observed: ' + a.observed_value),
          React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, 'Severity: ' + a.severity.toUpperCase() + ' · Status: ' + a.status),
          a.status !== 'reviewed' ? React.createElement(TouchableOpacity, {
            style: { backgroundColor: PRIMARY_COLOR, borderRadius: 8, padding: 10, alignItems: 'center', marginTop: 10 },
            onPress: function() { updateAnomaly({ id: a.id, data: { status: 'reviewed', reviewed_at: new Date().toISOString() } }).then(refetch); },
            componentId: 'review-anomaly-' + a.id
          }, React.createElement(Text, { style: { color: '#FFFFFF', fontWeight: '700', fontSize: 12 } }, 'MARK REVIEWED')) : null
        );
      })
    )
  );
};
// @end:AnomaliesScreen

// @section:OfficerTabNavigator @depends:[CommandCentreScreen,RecommendationsScreen,LoadBalancerScreen,SimulatorScreen,AnomaliesScreen,TransportOptimizationScreen,WarehouseManagementScreen,EmergencyProcurementScreen,PolicySimulatorScreen,OfficerAssistantScreen]
var OfficerTabNavigator = function() {
  var insets = useSafeAreaInsets();
  return React.createElement(Tab.Navigator, {
    screenOptions: function(routeProps) {
      var routeName = routeProps.route.name;
      return {
        headerShown: false,
        tabBarItemStyle: { padding: 0 },
        tabBarActiveTintColor: PRIMARY_COLOR,
        tabBarInactiveTintColor: TEXT_SECONDARY,
        tabBarStyle: { position: 'absolute', bottom: 0, height: Platform.OS === 'web' ? TAB_MENU_HEIGHT : TAB_MENU_HEIGHT + insets.bottom, paddingBottom: 0, borderTopWidth: 0, backgroundColor: '#FFFFFF' },
        tabBarIcon: function(iconProps) {
          var iconMap = { Command: 'speedometer', Recommendations: 'bulb', LoadBalancer: 'git-compare', Simulator: 'flask', Anomalies: 'warning', Transport: 'car', Warehouse: 'archive', Emergency: 'alert-circle', Policy: 'settings', Assistant: 'chatbubble-ellipses' };
          return React.createElement(Ionicons, { name: iconMap[routeName] || 'ellipse', size: 22, color: iconProps.color });
        }
      };
    }
  },
    React.createElement(Tab.Screen, { name: 'Command', component: CommandCentreScreen }),
    React.createElement(Tab.Screen, { name: 'Recommendations', component: RecommendationsScreen }),
    React.createElement(Tab.Screen, { name: 'LoadBalancer', component: LoadBalancerScreen }),
    React.createElement(Tab.Screen, { name: 'Simulator', component: SimulatorScreen }),
    React.createElement(Tab.Screen, { name: 'Anomalies', component: AnomaliesScreen }),
    React.createElement(Tab.Screen, { name: 'Transport', component: TransportOptimizationScreen }),
    React.createElement(Tab.Screen, { name: 'Warehouse', component: WarehouseManagementScreen }),
    React.createElement(Tab.Screen, { name: 'Emergency', component: EmergencyProcurementScreen }),
    React.createElement(Tab.Screen, { name: 'Policy', component: PolicySimulatorScreen }),
    React.createElement(Tab.Screen, { name: 'Assistant', component: OfficerAssistantScreen })
  );
};
// @end:OfficerTabNavigator

// @section:AdminOverviewScreen @depends:[ThemeContext,constants,styles]
var AdminOverviewScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreStorage[0];
  var roleState = useStorage('activeRole', null);
  var setRole = roleState[1];

  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var avgLoad = Math.round(CENTRES.reduce(function(s, c) { return s + centreLoads[c.id].current; }, 0) / CENTRES.length);

  var handleSwitchRole = function() {
    setRole(null);
    // User stays logged in, just role is cleared - will show RoleSelectScreen
  };

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'National Overview'),
      React.createElement(TouchableOpacity, { onPress: handleSwitchRole, componentId: 'admin-switch-role' },
        React.createElement(Ionicons, { name: 'swap-horizontal', size: 20, color: '#FFFFFF' })
      )
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: [styles.card, { marginBottom: 12 }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 8 } }, 'BEFORE vs AFTER AI OPTIMIZATION'),
        React.createElement(View, { style: { flexDirection: 'row' } },
          React.createElement(View, { style: { flex: 1 } },
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'BEFORE'),
            React.createElement(Text, { style: { fontSize: 18, fontWeight: '900', color: '#DC2626' } }, '82 min'),
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'Centre A 96% · HIGH')
          ),
          React.createElement(View, { style: { flex: 1 } },
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'AFTER'),
            React.createElement(Text, { style: { fontSize: 18, fontWeight: '900', color: '#16A34A' } }, '34 min'),
            React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, 'Centre A 71% · LOW')
          )
        ),
        React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: PRIMARY_COLOR, marginTop: 8 } }, '58.5% reduction in average waiting time')
      ),
      React.createElement(Text, { style: { fontSize: 15, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 8 } }, 'System Averages'),
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap', marginHorizontal: -6 } },
        [{ label: 'Avg Load', value: avgLoad + '%' }, { label: 'Active Centres', value: String(CENTRES.length) }].map(function(k, idx) {
          return React.createElement(View, { key: String(idx), style: { width: '50%', padding: 6 } },
            React.createElement(View, { style: [styles.card, { padding: 14 }] },
              React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, k.label),
              React.createElement(Text, { style: { fontSize: 20, fontWeight: '900', color: theme.colors.textPrimary, marginTop: 4 } }, k.value)
            )
          );
        })
      )
    )
  );
};
// @end:AdminOverviewScreen

// @section:AdminCentresScreen @depends:[ThemeContext,constants,styles]
var AdminCentresScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var centreLoads = centreStorage[0];
  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Centres')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      CENTRES.map(function(c) {
        var st = centreLoads[c.id];
        return React.createElement(View, { key: c.id, style: [styles.card, { marginBottom: 10 }] },
          React.createElement(Text, { style: { fontSize: 14, fontWeight: '700', color: theme.colors.textPrimary } }, c.name),
          React.createElement(Text, { style: { fontSize: 12, color: theme.colors.textSecondary, marginTop: 4 } }, c.location + ' · Capacity ' + c.dailyCapacity + ' · Counters ' + c.counters + ' · Staff ' + c.staff),
          React.createElement(Text, { style: { fontSize: 12, color: loadColor(st.current), marginTop: 4, fontWeight: '700' } }, 'Load ' + st.current + '% · Predicted ' + st.predicted + '% · Queue ' + st.queue)
        );
      })
    )
  );
};
// @end:AdminCentresScreen

// @section:AdminAnalyticsScreen @depends:[ThemeContext,BarChartSimple,styles]
var AdminAnalyticsScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var filterState = useState('7d');
  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var FILTERS = [{ id: 'today', label: 'Today' }, { id: '7d', label: '7 Days' }, { id: '30d', label: '30 Days' }];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Analytics')
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(View, { style: { flexDirection: 'row', marginBottom: 14 } },
        FILTERS.map(function(f) {
          var active = filterState[0] === f.id;
          return React.createElement(TouchableOpacity, { key: f.id, onPress: function() { filterState[1](f.id); }, style: { paddingVertical: 6, paddingHorizontal: 12, borderRadius: 16, backgroundColor: active ? PRIMARY_COLOR : '#EEF2E9', marginRight: 8 }, componentId: 'filter-' + f.id },
            React.createElement(Text, { style: { color: active ? '#FFFFFF' : theme.colors.textPrimary, fontSize: 12, fontWeight: '700' } }, f.label)
          );
        })
      ),
      React.createElement(View, { style: [styles.card, { marginBottom: 12 }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 4 } }, 'Crop Demand Forecast'),
        React.createElement(BarChartSimple, { chartId: 'crop-demand', data: [
          { label: 'Paddy', value: 380, color: PRIMARY_COLOR }, { label: 'Wheat', value: 210, color: ACCENT_COLOR }, { label: 'Maize', value: 145, color: '#7C9E5B' }, { label: 'Other', value: 90, color: '#B7C99A' }
        ] })
      ),
      React.createElement(View, { style: [styles.card, { marginBottom: 12 }] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 4 } }, 'Centre Utilization'),
        React.createElement(BarChartSimple, { chartId: 'centre-util', data: [
          { label: 'A', value: 87, color: loadColor(87) }, { label: 'B', value: 43, color: loadColor(43) }, { label: 'C', value: 61, color: loadColor(61) }
        ] })
      ),
      React.createElement(View, { style: [styles.card] },
        React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary, marginBottom: 4 } }, 'Recommendation Acceptance Rate'),
        React.createElement(BarChartSimple, { chartId: 'reco-accept', data: [{ label: 'Approved', value: 78, color: '#16A34A' }, { label: 'Rejected', value: 22, color: '#DC2626' }] })
      )
    )
  );
};
// @end:AdminAnalyticsScreen

// @section:AdminAuditLogsScreen @depends:[ThemeContext,styles]
var AdminAuditLogsScreen = function(props) {
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var { data: logs, loading } = useQuery('audit_logs', {}, { column: 'created_at', ascending: false });
  var scrollBottomPadding = Platform.OS === 'web' ? WEB_TAB_MENU_PADDING : (TAB_MENU_HEIGHT + insets.bottom + SCROLL_EXTRA_PADDING);
  var list = (logs && logs.length > 0) ? logs : [];

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20 } },
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 18, fontWeight: '800' } }, 'Audit Logs')
    ),
    loading ? React.createElement(ActivityIndicator, { style: { flex: 1 }, componentId: 'audit-loading' }) :
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      list.length === 0 ? React.createElement(Text, { style: { color: theme.colors.textSecondary, textAlign: 'center', marginTop: 20 } }, 'No audit events yet — approve or reject a recommendation to generate one') :
      list.map(function(log) {
        return React.createElement(View, { key: log.id, style: [styles.card, { marginBottom: 8 }] },
          React.createElement(Text, { style: { fontSize: 12, fontWeight: '700', color: theme.colors.textPrimary } }, log.action),
          React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary, marginTop: 2 } }, 'Entity: ' + log.entity_type + ' · ' + new Date(log.created_at).toLocaleString())
        );
      })
    )
  );
};
// @end:AdminAuditLogsScreen

// @section:AdminTabNavigator @depends:[AdminOverviewScreen,AdminCentresScreen,AdminAnalyticsScreen,AdminAuditLogsScreen]
var AdminTabNavigator = function() {
  var insets = useSafeAreaInsets();
  return React.createElement(Tab.Navigator, {
    screenOptions: function(routeProps) {
      var routeName = routeProps.route.name;
      return {
        headerShown: false,
        tabBarItemStyle: { padding: 0 },
        tabBarActiveTintColor: PRIMARY_COLOR,
        tabBarInactiveTintColor: TEXT_SECONDARY,
        tabBarStyle: { position: 'absolute', bottom: 0, height: Platform.OS === 'web' ? TAB_MENU_HEIGHT : TAB_MENU_HEIGHT + insets.bottom, paddingBottom: 0, borderTopWidth: 0, backgroundColor: '#FFFFFF' },
        tabBarIcon: function(iconProps) {
          var iconMap = { Overview: 'globe', Centres: 'business', Analytics: 'bar-chart', AuditLogs: 'document-text' };
          return React.createElement(Ionicons, { name: iconMap[routeName] || 'ellipse', size: 22, color: iconProps.color });
        }
      };
    }
  },
    React.createElement(Tab.Screen, { name: 'Overview', component: AdminOverviewScreen }),
    React.createElement(Tab.Screen, { name: 'Centres', component: AdminCentresScreen }),
    React.createElement(Tab.Screen, { name: 'Analytics', component: AdminAnalyticsScreen }),
    React.createElement(Tab.Screen, { name: 'AuditLogs', component: AdminAuditLogsScreen })
  );
};
// @end:AdminTabNavigator

// @section:CentreDetailScreen @depends:[ThemeContext,constants,styles]
var CentreDetailScreen = function(props) {
  var route = props.route;
  var navigation = props.navigation;
  var themeCtx = useTheme();
  var theme = themeCtx.theme;
  var insets = useSafeAreaInsets();
  var centreId = route && route.params ? route.params.centreId : 'centre-a';
  var centre = CENTRES.filter(function(c) { return c.id === centreId; })[0] || CENTRES[0];
  var centreStorage = useStorage('centreState', DEFAULT_CENTRE_STATE);
  var st = centreStorage[0][centre.id];

  var counters = [
    { id: 1, minutes: 12 }, { id: 2, minutes: 28 }, { id: 3, minutes: 51 }
  ];
  var counterColor = function(m) { return m < 20 ? '#16A34A' : (m < 40 ? '#F59E0B' : '#DC2626'); };
  var scrollBottomPadding = insets.bottom + SCROLL_EXTRA_PADDING;

  return React.createElement(View, { style: { flex: 1, backgroundColor: theme.colors.background } },
    React.createElement(StatusBar, { backgroundColor: PRIMARY_COLOR, barStyle: 'light-content' }),
    React.createElement(View, { style: { backgroundColor: PRIMARY_COLOR, paddingTop: insets.top + 12, paddingBottom: 14, paddingHorizontal: 20, flexDirection: 'row', alignItems: 'center' } },
      React.createElement(TouchableOpacity, { onPress: function() { navigation.goBack(); }, style: { marginRight: 12 }, componentId: 'centre-detail-back' },
        React.createElement(Ionicons, { name: 'arrow-back', size: 22, color: '#FFFFFF' })
      ),
      React.createElement(Text, { style: { color: '#FFFFFF', fontSize: 17, fontWeight: '800' } }, centre.name.toUpperCase())
    ),
    React.createElement(ScrollView, { contentContainerStyle: { padding: 16, paddingBottom: scrollBottomPadding } },
      React.createElement(Text, { style: { fontSize: 14, fontWeight: '800', color: theme.colors.textPrimary, marginBottom: 8 } }, 'Counters'),
      counters.map(function(c) {
        return React.createElement(View, { key: String(c.id), style: [styles.card, { marginBottom: 8, flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' }] },
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '700', color: theme.colors.textPrimary } }, 'Counter ' + c.id),
          React.createElement(Text, { style: { fontSize: 13, fontWeight: '800', color: counterColor(c.minutes) } }, c.minutes + ' min')
        );
      }),
      React.createElement(View, { style: { flexDirection: 'row', flexWrap: 'wrap', marginHorizontal: -6, marginTop: 6 } },
        [
          { label: 'Queue', value: st.queue + ' farmers' },
          { label: 'Capacity', value: String(centre.dailyCapacity) },
          { label: 'Current Load', value: st.current + '%' },
          { label: 'Predicted Load', value: st.predicted + '%' },
          { label: 'Processing Rate', value: '32/hour' }
        ].map(function(k, idx) {
          return React.createElement(View, { key: String(idx), style: { width: '50%', padding: 6 } },
            React.createElement(View, { style: [styles.card, { padding: 14 }] },
              React.createElement(Text, { style: { fontSize: 11, color: theme.colors.textSecondary } }, k.label),
              React.createElement(Text, { style: { fontSize: 15, fontWeight: '800', color: theme.colors.textPrimary, marginTop: 4 } }, k.value)
            )
          );
        })
      )
    )
  );
};
// @end:CentreDetailScreen

// @section:RoleBasedTabs @depends:[FarmerTabNavigator,OfficerTabNavigator,AdminTabNavigator]
var RoleBasedTabs = function() {
  var roleStorage = useStorage('activeRole', null);
  var role = roleStorage[0];
  if (role === 'officer') return React.createElement(OfficerTabNavigator);
  if (role === 'admin') return React.createElement(AdminTabNavigator);
  return React.createElement(FarmerTabNavigator);
};
// @end:RoleBasedTabs

// @section:AppStack @depends:[RoleBasedTabs,CentreDetailScreen]
var AppStack = function() {
  return React.createElement(Stack.Navigator, { screenOptions: { headerShown: false }, initialRouteName: 'MainTabs' },
    React.createElement(Stack.Screen, { name: 'MainTabs', component: RoleBasedTabs }),
    React.createElement(Stack.Screen, { name: 'CentreDetail', component: CentreDetailScreen, initialParams: { centreId: 'centre-a' } })
  );
};
// @end:AppStack

// @section:RootSwitcher @depends:[RoleSelectScreen,AppStack,LoginScreen]
var RootSwitcher = function() {
  var currentUserStorage = useStorage('currentUser', null);
  var currentUser = currentUserStorage[0];
  var roleStorage = useStorage('activeRole', null);
  var role = roleStorage[0];
  
  // If user is not logged in, show login screen
  if (!currentUser) {
    return React.createElement(LoginScreen);
  }
  
  // If user is logged in but no role selected, show role selection
  if (!role) {
    return React.createElement(RoleSelectScreen);
  }
  
  // User is logged in and role is selected, show app
  return React.createElement(AppStack);
};
// @end:RootSwitcher

// @section:styles @depends:[theme]
var styles = StyleSheet.create({
  card: {
    backgroundColor: CARD_COLOR,
    borderRadius: 14,
    padding: 16,
    shadowColor: '#000',
    shadowOpacity: 0.06,
    shadowRadius: 6,
    shadowOffset: { width: 0, height: 2 },
    elevation: 2
  },
  label: {
    fontSize: 12,
    fontWeight: '700',
    color: TEXT_SECONDARY,
    marginBottom: 6,
    marginTop: 10
  },
  input: {
    borderWidth: 1,
    borderColor: '#D1D5DB',
    borderRadius: 8,
    padding: 12,
    fontSize: 14,
    color: TEXT_PRIMARY,
    backgroundColor: '#FFFFFF'
  }
});
// @end:styles

// @section:return @depends:[ThemeProvider,RootSwitcher]
return React.createElement(ThemeProvider, null,
  React.createElement(View, { style: { flex: 1, width: '100%', height: '100%' } },
    React.createElement(RootSwitcher)
  )
);
// @end:return
};
return ComponentFunction;
