# Network Connection Context Provider

### Used Libraries:
- @gorhom/bottom-sheet
- @react-native-community/netinfo (redundant use with isThereConnection api function)
- react-native-vector-icons/MaterialIcons
- react-native-gesture-handler

```tsx
import { translate } from '@assets/locales/translate'
import { Colors, CustomEventHandle } from '@constants'
import BottomSheet, {
  BottomSheetBackdrop,
  useBottomSheetModal
} from '@gorhom/bottom-sheet'
import { useNetInfo } from '@react-native-community/netinfo'
import { isThereConnection } from '@services/api'
import { Spacer, Text } from '@stylescomponents'
import React, {
  createContext,
  useCallback,
  useContext,
  useEffect,
  useMemo,
  useState
} from 'react'
import { DeviceEventEmitter, View } from 'react-native'
import { TouchableOpacity } from 'react-native-gesture-handler'
import Icon from 'react-native-vector-icons/MaterialIcons'

const ConnectionContext = createContext<NetworkConnectionContextData>(
  {} as NetworkConnectionContextData
)

const NetworkConnectionProvider: React.FC = ({ children }) => {
  const [show, setShow] = useState<boolean>(false)
  const { dismissAll } = useBottomSheetModal()

  const netInfo = useNetInfo()
  const [connected, setConnected] = useState<boolean>(
    netInfo.isConnected || true
  )

  const snapPoints = useMemo(() => ['70%'], [])

  useEffect(() => {
    const withoutConnectionListener = DeviceEventEmitter.addListener(
      CustomEventHandle.WITHOUT_NETWORK_CONNECTION,
      () => showModalWithoutConnection()
    )

    return () => {
      if (withoutConnectionListener) {
        withoutConnectionListener.remove()
      }
    }
  }, [])

  useEffect(() => {
    if (connected) {
      DeviceEventEmitter.emit(CustomEventHandle.RECONECTED_NETWORK_CONNECTION)
    }
  }, [connected])

  useEffect(() => {
    if (netInfo.isConnected != null && netInfo.isConnected != connected) {
      if (!netInfo.isConnected) {
        dismissAll()
      }

      setShow(!netInfo.isConnected)
      setConnected(netInfo.isConnected)
    }
  }, [netInfo])

  const renderBackdrop = useCallback(
    (props) => (
      <BottomSheetBackdrop
        pressBehavior="none"
        disappearsOnIndex={-1}
        appearsOnIndex={0}
        {...props}
      />
    ),
    []
  )

  const showModalWithoutConnection = useCallback(async () => {
    if (!show) {
      dismissAll()
      setShow(true)
    }
  }, [show])

  const tryAgain = useCallback(async () => {
    setShow(false)

    if (!(await isThereConnection())) {
      setShow(true)
      return
    }

    DeviceEventEmitter.emit(CustomEventHandle.RETRY_NETWORK_CONNECTION)
  }, [])

  return (
    <ConnectionContext.Provider value>
      {children}

      {show && (
        <BottomSheet
          handleIndicatorStyle={{
            backgroundColor: Colors.LIGHT
          }}
          enableOverDrag={false}
          enablePanDownToClose={false}
          backdropComponent={renderBackdrop}
          snapPoints={snapPoints}
        >
          <View
            style={{
              alignItems: 'center',
              marginTop: 50
            }}
          >
            <Icon name="wifi-off" size={50} color={Colors.DEFAULT} />

            <Spacer heigth={40} />
            <Text style={{ maxWidth: 300 }}>
              {translate('connection.noConnectionMessage')}
            </Text>

            <Spacer />
            <TouchableOpacity onPress={() => tryAgain()}>
              <Text fontFamily="bold" textDecorationLine="underline">
                {translate('default.tryAgain')}
              </Text>
            </TouchableOpacity>
          </View>
        </BottomSheet>
      )}
    </ConnectionContext.Provider>
  )
}

function useConnectionContext(): NetworkConnectionContextData {
  const context = useContext(ConnectionContext)
  if (!context) {
    throw new Error('useContext must be used within an provider')
  }

  return context
}

export { NetworkConnectionProvider, useConnectionContext }

```