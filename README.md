
## GlobalPOS Integration
### Broadcast Receiver for GlobalPOS

The GlobalPOS scanner broadcasts scan events using the **Intent action** `"com.android.scanservice.scancontext"`. This action is accompanied by the scanned data in the **"Scan_context"** key.

### Key Elements:

-   **Action**: `com.android.scanservice.scancontext`
-   **Data Key**: `Scan_context` (the scanned data string)


### Code Implementation
Below is the implementation for handling scan data from GlobalPOS devices.
#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        private const val SCAN_ACTION = "com.android.scanservice.scancontext"
    }

    private lateinit var textView: TextView
    private lateinit var scanReceiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        textView = findViewById(R.id.textView)

        // Initialize BroadcastReceiver
        scanReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                if (SCAN_ACTION == intent?.action) {
                    // Get the scanned data
                    val scannedData = intent?.getStringExtra("Scan_context")
                    handleScannedData(scannedData)
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        // Register the receiver
        val filter = IntentFilter(SCAN_ACTION)
        registerReceiver(scanReceiver, filter)
    }

    override fun onPause() {
        super.onPause()
        // Unregister the receiver
        unregisterReceiver(scanReceiver)
    }

    // Method to handle the scanned data
    private fun handleScannedData(data: String?) {
        data?.let {
            Log.d("GlobalPOS", "Scanned Data: $it")
            textView.text = it
        }
    }
}
```

## Urovo Integration

### Broadcast Receiver for Urovo

The Urovo scanner broadcasts scan events using the **Intent action** `"urovo.rcv.message"`. The scanned data is provided as a byte array under the key `"barocode"`, and the barcode type is included in the intent as a byte value under the key `"barcodeType"`.

### Key Elements:

-   **Action**: `urovo.rcv.message`
-   **Data Keys**:
    -   `barocode` (the scanned data as a byte array)
    -   `length` (integer value representing the length of the scanned data)
    -   `barcodeType` (the type of barcode as a byte value)

### Code Implementation

Below is the implementation for handling scan data from Urovo devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        private const val SCAN_ACTION = "urovo.rcv.message"
    }

    private lateinit var textView: TextView
    private lateinit var scanReceiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        textView = findViewById(R.id.textView)

        // Initialize BroadcastReceiver
        scanReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                if (SCAN_ACTION == intent?.action) {
                    // Get the scanned barcode as byte array
                    val barcodeBytes = intent.getByteArrayExtra("barocode")
                    val length = intent.getIntExtra("length", 0)
                    val barcodeType = intent.getByteExtra("barcodeType", 0)

                    if (barcodeBytes != null) {
                        // Convert byte array to string using the length of the data
                        val scannedData = String(barcodeBytes, 0, length)
                        val barcodeTypeStr = getBarcodeType(barcodeType)
                        handleScannedData(scannedData, barcodeTypeStr)
                    }
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        // Register the receiver
        val filter = IntentFilter(SCAN_ACTION)
        registerReceiver(scanReceiver, filter)
    }

    override fun onPause() {
        super.onPause()
        // Unregister the receiver
        unregisterReceiver(scanReceiver)
    }

    // Method to handle the scanned data
    private fun handleScannedData(data: String, barcodeType: String) {
        Log.d("Urovo", "Scanned Data: $data, Barcode Type: $barcodeType")
        textView.text = "Data: $data, Type: $barcodeType"
    }

    // Convert barcode type byte to a string representation
    private fun getBarcodeType(barcodeType: Byte): String {
        return when (barcodeType) {
            3.toByte(), 106.toByte() -> "CODE128"
            15.toByte(), 50.toByte(), 73.toByte(), 121.toByte(), 123.toByte(), 125.toByte() -> "GS1"
            17.toByte(), 114.toByte() -> "PDF417"
            27.toByte(), 119.toByte() -> "DATA_MATRIX"
            100.toByte() -> "EAN"
            else -> "UNKNOWN"
        }
    }
}
```

## Urovo Integration (Alternate Variant)

### Broadcast Receiver for Urovo

The Urovo scanner broadcasts scan events using the **Intent action** `"android.intent.ACTION_DECODE_DATA"`. The scanned data is provided as a string under the key `"barcode_string"`, and the barcode type is provided as a byte value under the key `"barcodeType"`.

### Key Elements:

-   **Action**: `android.intent.ACTION_DECODE_DATA`
-   **Data Keys**:
    -   `barcode_string` (the scanned data as a string)
    -   `barcodeType` (the type of barcode as a byte value)

### Code Implementation

Below is the Kotlin implementation for handling scan data from Urovo devices in this alternate variant.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        private const val SCAN_ACTION = "android.intent.ACTION_DECODE_DATA"
        private const val BARCODE_STRING_KEY = "barcode_string"
        private const val BARCODE_TYPE_KEY = "barcodeType"
    }

    private lateinit var textView: TextView
    private lateinit var scanReceiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        textView = findViewById(R.id.textView)

        // Initialize BroadcastReceiver
        scanReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                if (SCAN_ACTION == intent?.action) {
                    // Get the scanned barcode string
                    val barcodeData = intent.getStringExtra(BARCODE_STRING_KEY)
                    // Get the barcode type
                    val barcodeType = intent.getByteExtra(BARCODE_TYPE_KEY, 0)

                    if (barcodeData != null) {
                        val barcodeTypeStr = getBarcodeType(barcodeType)
                        handleScannedData(barcodeData, barcodeTypeStr)
                    }
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        // Register the receiver
        val filter = IntentFilter(SCAN_ACTION)
        registerReceiver(scanReceiver, filter)
    }

    override fun onPause() {
        super.onPause()
        // Unregister the receiver
        unregisterReceiver(scanReceiver)
    }

    // Method to handle the scanned data
    private fun handleScannedData(data: String, barcodeType: String) {
        Log.d("Urovo", "Scanned Data: $data, Barcode Type: $barcodeType")
        textView.text = "Data: $data, Type: $barcodeType"
    }

    // Convert barcode type byte to a string representation
    private fun getBarcodeType(barcodeType: Byte): String {
        return when (barcodeType) {
            8.toByte() -> "CODE128"
            17.toByte(), 19.toByte() -> "GS1"
            42.toByte(), 11.toByte(), 12.toByte() -> "EAN"
            22.toByte() -> "PDF417"
            23.toByte() -> "DATA_MATRIX"
            else -> "UNKNOWN"
        }
    }
}
```

## Vioteh Integration

### Overview

The **Vioteh** scanner integration uses the `SerialPort` API for communication with the scanner and a handler to process incoming barcode data. The scanner interacts with hardware components, including the GPIO for managing the scan trigger and power. Scanned data is received via the serial port and processed using the `Handler` message system.

### Key Elements:

-   **Serial Port Communication**: The scanner communicates via the serial port using the `SerialPort` class.
-   **Handler**: The barcode data is handled by a `Handler` that listens for messages containing scanned data.
-   **Scanner Control**: The scanner is controlled by the `ScanGpio` class to manage scanning and power.

### Code Implementation

Below is the Kotlin implementation for handling the **Vioteh** scanner integration.

#### Kotlin Implementation:
https://github.com/mango319/DeviceSDK
```kotlin
package com.example.viotehscanner

import android.content.Context
import android.os.Bundle
import android.os.Handler
import android.os.Message
import android.util.Log
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.smartdevicesdk.device.DeviceInfo
import com.smartdevicesdk.device.DeviceManage
import com.smartdevicesdk.scanner.ScanGpio
import com.smartdevicesdk.utils.HandlerMessage
import java.nio.charset.StandardCharsets
import android.serialport.api.SerialPort
import android.serialport.api.SerialPortDataReceived

class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private lateinit var startScanButton: Button
    private var scannerHelper: ScannerHelper? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView and Button
        textView = findViewById(R.id.textView)
        startScanButton = findViewById(R.id.startScanButton)

        // Set click listener for the scan button
        startScanButton.setOnClickListener {
            startScan()
        }
    }

    override fun onResume() {
        super.onResume()
        // Connect to the scanner when the activity resumes
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        // Disconnect from the scanner when the activity pauses
        disconnectScanner()
    }

    // Connect to the scanner
    private fun connectScanner(): Boolean {
        return try {
            val deviceInfo: DeviceInfo = DeviceManage.getDevInfo("PDA3501")
            val handler = object : Handler() {
                override fun handleMessage(msg: Message) {
                    if (msg.what == HandlerMessage.SCANNER_DATA_MSG && msg.obj != null) {
                        // Get the scanned data and clean it
                        val barcode = msg.obj.toString().replace("\r", "").replace("\n", "").replace("\u0000", "")
                        handleScannedData(barcode)
                    }
                }
            }
            scannerHelper = ScannerHelper(this, deviceInfo.scannerSerialport, deviceInfo.scannerBaudrate, handler)
            true
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
            false
        }
    }

    // Disconnect from the scanner
    private fun disconnectScanner() {
        scannerHelper?.close()
    }

    // Start the scan
    private fun startScan() {
        try {
            scannerHelper?.scan()
        } catch (e: Exception) {
            Log.e("MainActivity", "Error starting scan: ${e.message}")
        }
    }

    // Handle the scanned data
    private fun handleScannedData(data: String) {
        Log.d("MainActivity", "Scanned Data: $data")
        runOnUiThread {
            textView.text = "Scanned Data: $data"
        }
    }

    // Helper class to manage the scanner
    private class ScannerHelper(context: Context, serialPortPath: String, baudrate: Int, private val handler: Handler) {

        private val serialPort: SerialPort = SerialPort(serialPortPath, baudrate)
        private val scanGpio = ScanGpio()
        private var barcode = ""

        init {
            serialPort.open()
            serialPort.setOnserialportDataReceived(object : SerialPortDataReceived {
                override fun onDataReceivedListener(data: ByteArray, length: Int) {
                    handleDataReceived(data, length)
                }
            })
            scanGpio.openPower()
        }

        // Handle data received from the scanner
        private fun handleDataReceived(data: ByteArray, length: Int) {
            if (length == 0) return

            val receivedData = String(data, 0, length, StandardCharsets.UTF_8)
            barcode += receivedData
            if (receivedData.contains("\n")) {
                val msg = handler.obtainMessage(HandlerMessage.SCANNER_DATA_MSG, barcode)
                handler.sendMessage(msg)
                barcode = ""
            }
        }

        // Close the scanner
        fun close() {
            scanGpio.closePower()
            serialPort.closePort()
            scanGpio.closeScan()
        }

        // Trigger the scan
        fun scan() {
            scanGpio.openScan()
            try {
                Thread.sleep(50) // Delay to ensure scan is triggered correctly
            } catch (e: InterruptedException) {
                e.printStackTrace()
            }
        }
    }
}
```

## Proton Integration

### Overview

The **Proton** scanner uses the **ReaderManager** to manage the scanning process. The integration primarily operates in broadcast mode, where scan events are broadcasted using the **Intent action** `"com.android.server.scannerservice.broadcast"`. The scanned data is received via the broadcast intent and processed by the app. The ReaderManager is responsible for managing the scan hardware and configuration settings.

### Key Elements:

-   **Broadcast Receiver**: The Proton scanner broadcasts scan results using the action `"com.android.server.scannerservice.broadcast"`.
-   **ReaderManager**: This component manages the scanner hardware, including enabling scan keys, setting output mode, and handling end characters.
-   **Data Keys**:
    -   `scannerdata` (the scanned barcode data as a string)
    -   `barcodeType` (the type of barcode as a byte value)

### Code Implementation

Below is the Kotlin implementation for handling scan data from **Proton** devices.

#### Kotlin Implementation:
https://github.com/DelphiTeacher/OrangeFreeSDK/blob/master/SupoinPDA_Scan/PDA(android)%E5%BC%80%E5%8F%91%E5%8C%85/jar%E5%8C%85/ReaderManager.jar
```kotlin
package com.example.protonscanner

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.Bundle
import android.util.Log
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.android.scanner.impl.ReaderManager

class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private var readerManager: ReaderManager? = null
    private val SCAN_ACTION = "com.android.server.scannerservice.broadcast"

    private val scanReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            if (intent?.action == SCAN_ACTION) {
                readerManager?.stopScanAndDecode() // Stop scanning after receiving data
                val barcodeData = intent.getStringExtra("scannerdata") ?: ""
                val barcodeType = intent.getByteExtra("barcodeType", 0)
                handleScannedData(barcodeData, barcodeType)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scan data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect the scanner and configure it
    private fun connectScanner(): Boolean {
        return try {
            readerManager = ReaderManager.getInstance()
            readerManager?.let { rm ->
                if (!rm.GetActive()) rm.SetActive(true)
                if (!rm.isEnableScankey()) rm.setEnableScankey(true)
                if (rm.getOutPutMode() != 2) rm.setOutPutMode(2)
                if (rm.getEndCharMode() != 3) rm.setEndCharMode(3)

                // Register a BroadcastReceiver to receive scanner data
                val filter = IntentFilter(SCAN_ACTION)
                registerReceiver(scanReceiver, filter)
            }
            true
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
            false
        }
    }

    // Disable the scanner and unregister BroadcastReceiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(scanReceiver)
            readerManager?.Release()
            readerManager = null
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Process the received scan data and update the UI
    private fun handleScannedData(data: String, barcodeType: Byte) {
        Log.d("MainActivity", "Scanned Data: $data, Barcode Type: $barcodeType")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $barcodeType"
        }
    }
}
```
## Caribe Integration

#### Broadcast Receiver for Caribe

The **Caribe** scanner broadcasts scan events using the **Intent action** `"scan.rcv.message"` and **PL-40 action** `"com.action.SCAN_RESULT"`. The events contain scanned data in byte arrays, which can be processed into readable text.

#### Key Elements:

-   **Action**: `scan.rcv.message` and `com.action.SCAN_RESULT`
-   **Data Keys**:
    -   `barocode` (scanned barcode data in bytes)
    -   `length` (length of the barcode data)
    -   `aimid` (AIMID for identifying barcode type)

#### Code Implementation

Below is the implementation for handling scan data from **Caribe** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private var scanDevice: ScanDevice? = null
    private val SCAN_ACTION = "scan.rcv.message"
    private val SCAN_ACTION_PL_40 = "com.action.SCAN_RESULT"

    // BroadcastReceiver for scanning action
    private val scanReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            try {
                val barcodeBytes = intent?.getByteArrayExtra("barocode")
                val length = intent?.getIntExtra("length", 0) ?: 0
                val aimidBytes = intent?.getByteArrayExtra("aimid")
                val barcode = barcodeBytes?.let { String(it, 0, length) } ?: "Unknown"
                val aimid = aimidBytes?.let { ByteString.of(it).toString() } ?: "UNKNOWN"
                val barcodeType = getTypeBarcode(aimid)

                handleScannedData(barcode, barcodeType)

                scanDevice?.stopScan()
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    // BroadcastReceiver for PL-40 scan action
    private val scanReceiverPl40 = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            val scanContextBytes = intent?.getByteArrayExtra("scanContext")
            val scanContext = scanContextBytes?.let { String(it, 0, it.size - 3) } ?: "Unknown"
            handleScannedData(scanContext, "PL-40")
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register broadcast receivers
    private fun connectScanner() {
        try {
            val filter = IntentFilter(SCAN_ACTION)
            val filterPl40 = IntentFilter(SCAN_ACTION_PL_40)
            registerReceiver(scanReceiver, filter)
            registerReceiver(scanReceiverPl40, filterPl40)
            tryEnableDriver()
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister broadcast receivers
    private fun disconnectScanner() {
        try {
            unregisterReceiver(scanReceiver)
            unregisterReceiver(scanReceiverPl40)
            scanDevice?.closeScan()
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Enable the scan driver
    private fun tryEnableDriver() {
        try {
            scanDevice = ScanDevice()
            scanDevice?.openScan()
        } catch (t: Throwable) {
            scanDevice = null
        }
    }

    // Handle the scanned data and update UI
    private fun handleScannedData(data: String, type: String) {
        Log.d("MainActivity", "Scanned Data: $data, Type: $type")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $type"
        }
    }

    // Map the barcode type based on AIMID
    private fun getTypeBarcode(aimid: String): String {
        return when (aimid) {
            "[text=]d1]" -> "DATA_MATRIX"
            "[text=]d2]" -> "GS1_DATA_MATRIX"
            "[text=]E0]", "[text=]E4]" -> "EAN"
            "[text=]C0]" -> "CODE128"
            "[text=]L2]" -> "PDF417"
            "[text=]e0]" -> "GS1"
            else -> "UNKNOWN"
        }
    }
}
```
## NewLandMT65/90/N7 Integration

#### Broadcast Receiver for NewLandMT65/90/N7

The **NewLandMT65/90/N7** scanner broadcasts scan events using the **Intent action** `"com.android.action.SEND_SCAN_RESULT"`. The events contain the scanned data in byte arrays, which can be processed into a readable string. The barcode type is identified based on the received integer value.

#### Key Elements:

-   **Action**: `com.android.action.SEND_SCAN_RESULT`
-   **Data Keys**:
    -   `scan_result_one_bytes` (scanned barcode data in bytes)
    -   `SCAN_BARCODE_TYPE` (barcode type identifier)

#### Code Implementation

Below is the implementation for handling scan data from **NewLandMT65/90/N7** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private val SCAN_ACTION = "com.android.action.SEND_SCAN_RESULT"

    // BroadcastReceiver to handle scan result
    private val resultReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            if (SCAN_ACTION == intent?.action) {
                val byteArrayExtra = intent.getByteArrayExtra("scan_result_one_bytes")
                val barcodeTypeInt = intent.getIntExtra("SCAN_BARCODE_TYPE", -1)
                
                byteArrayExtra?.let {
                    val barcode = String(it, StandardCharsets.UTF_8)
                    val barcodeType = getBarcodeType(barcodeTypeInt)

                    // If GS1 barcode, replace '~' with ASCII 29 character
                    val formattedBarcode = if (barcodeType == "GS1") {
                        barcode.replace('~', 29.toChar())
                    } else {
                        barcode
                    }
                    
                    handleScannedData(formattedBarcode, barcodeType)
                }
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register the broadcast receiver
    private fun connectScanner() {
        try {
            val filter = IntentFilter(SCAN_ACTION)
            registerReceiver(resultReceiver, filter)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister the broadcast receiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(resultReceiver)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Handle the scanned data and update UI
    private fun handleScannedData(data: String, type: String) {
        Log.d("MainActivity", "Scanned Data: $data, Type: $type")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $type"
        }
    }

    // Map the barcode type based on the integer value from the scan result
    private fun getBarcodeType(typeInt: Int): String {
        return when (typeInt) {
            2 -> "CODE128"
            3, 5 -> "GS1"
            8 -> "EAN"
            256 -> "PDF417"
            261 -> "DATA_MATRIX"
            else -> "UNKNOWN"
        }
    }
}
```

## Unitech Integration

#### Broadcast Receiver for Unitech

The **Unitech** scanner broadcasts scan events using the **Intent actions** `"unitech.scanservice.data"`, `"unitech.scanservice.databyte"`, `"unitech.scanservice.datalength"`, and `"unitech.scanservice.datatype"`. These actions contain scanned data in different formats (strings, bytes, types) and must be processed accordingly.

#### Key Elements:

-   **Actions**:
    -   `unitech.scanservice.data` (scanned data as a string)
    -   `unitech.scanservice.databyte` (scanned data as byte array)
    -   `unitech.scanservice.datalength` (length of scanned data)
    -   `unitech.scanservice.datatype` (barcode type identifier)
-   **Data Keys**:
    -   `"text"` (holds the scanned data or barcode type in various formats)

#### Code Implementation

Below is the implementation for handling scan data from **Unitech** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private val ACTION_RECEIVE_DATA = "unitech.scanservice.data"
    private val ACTION_RECEIVE_DATABYTES = "unitech.scanservice.databyte"
    private val ACTION_RECEIVE_DATATYPE = "unitech.scanservice.datatype"
    private val unitechBarcodeData = UnitechBarcodeData()

    // BroadcastReceiver to handle scan result
    private val dataReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            val action = intent?.action
            val extras = intent?.extras

            extras?.let {
                when (action) {
                    ACTION_RECEIVE_DATABYTES -> {
                        val byteArray = it.getByteArray("text")
                        byteArray?.let { bytes ->
                            val formatter = Formatter()
                            for (byte in bytes) {
                                formatter.format("%02X ", byte)
                            }
                        }
                    }
                    ACTION_RECEIVE_DATATYPE -> {
                        unitechBarcodeData.barcodeType = it.getInt("text")
                    }
                    ACTION_RECEIVE_DATA -> {
                        var barcode = it.getString("text") ?: ""
                        if (barcode.endsWith("\n")) {
                            barcode = barcode.substring(0, barcode.length - 1)
                        }
                        unitechBarcodeData.barcode = barcode
                    }
                }

                // When both barcode and barcodeType are available, process the data
                if (unitechBarcodeData.barcode.isNotEmpty() && unitechBarcodeData.barcodeType != -1) {
                    handleScannedData(
                        unitechBarcodeData.barcode,
                        getTypeBarcode(unitechBarcodeData.barcodeType)
                    )
                    unitechBarcodeData.reset() // Clear after processing
                }
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register the broadcast receiver
    private fun connectScanner() {
        try {
            disableScanToKey()
            val filter = IntentFilter().apply {
                addAction(ACTION_RECEIVE_DATA)
                addAction(ACTION_RECEIVE_DATABYTES)
                addAction(ACTION_RECEIVE_DATATYPE)
            }
            registerReceiver(dataReceiver, filter)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister the broadcast receiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(dataReceiver)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Disable scan-to-key feature
    private fun disableScanToKey() {
        val intent = Intent().apply {
            action = "unitech.scanservice.scan2key_setting"
            putExtra("scan2key", false)
        }
        sendBroadcast(intent)
    }

    // Handle the scanned data and update the UI
    private fun handleScannedData(data: String, type: String) {
        Log.d("MainActivity", "Scanned Data: $data, Type: $type")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $type"
        }
    }

    // Map the barcode type based on the integer value from the scan result
    private fun getTypeBarcode(typeInt: Int): String {
        return when (typeInt) {
            68, 100 -> "EAN"
            106 -> "CODE128"
            119 -> "DATA_MATRIX"
            114 -> "PDF417"
            73 -> "GS1"
            else -> "UNKNOWN"
        }
    }

    // Data class to hold barcode information
    data class UnitechBarcodeData(
        var barcode: String = "",
        var barcodeType: Int = -1
    ) {
        fun reset() {
            barcode = ""
            barcodeType = -1
        }
    }
}
```

## M3 Integration

#### Broadcast Receiver for M3

The **M3** scanner broadcasts scan events using the **Intent action** `"com.android.server.scannerservice.broadcast"`. The events contain the scanned data and the barcode type as strings.

#### Key Elements:

-   **Action**: `com.android.server.scannerservice.broadcast`
-   **Data Keys**:
    -   `m3scannerdata` (scanned barcode data as a string)
    -   `m3scanner_code_type` (barcode type as a string)

#### Code Implementation

Below is the implementation for handling scan data from **M3** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private val SCAN_ACTION = "com.android.server.scannerservice.broadcast"

    // BroadcastReceiver to handle scan result
    private val scanReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            val scannedData = intent?.getStringExtra("m3scannerdata")
            val barcodeTypeStr = intent?.getStringExtra("m3scanner_code_type")

            scannedData?.let {
                val barcodeType = getBarcodeType(barcodeTypeStr)
                handleScannedData(it, barcodeType)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register the broadcast receiver
    private fun connectScanner() {
        try {
            val filter = IntentFilter(SCAN_ACTION)
            registerReceiver(scanReceiver, filter)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister the broadcast receiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(scanReceiver)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Handle the scanned data and update UI
    private fun handleScannedData(data: String, type: String) {
        Log.d("MainActivity", "Scanned Data: $data, Type: $type")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $type"
        }
    }

    // Map the barcode type based on the string value from the scan result
    private fun getBarcodeType(typeStr: String?): String {
        return when (typeStr) {
            "EAN-128" -> "GS1"
            "DataMatrix" -> "DATA_MATRIX"
            "CODE128" -> "CODE128"
            "PDF-417" -> "PDF417"
            "QR Code" -> "QR_CODE"
            "EAN13" -> "EAN"
            else -> "UNKNOWN"
        }
    }
}
```

## Chainway Integration

#### Broadcast Receiver for Chainway

The **Chainway** scanner broadcasts scan events using the **Intent action** `"com.scanner.broadcast"`. The events contain the scanned data in the form of a string and the barcode type.

#### Key Elements:

-   **Action**: `com.scanner.broadcast`
-   **Data Keys**:
    -   `data` (scanned barcode data as a string)
    -   `CODE_TYPE` (barcode type as a string)

#### Code Implementation

Below is the implementation for handling scan data from **Chainway** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private val ACTION = "com.scanner.broadcast"

    // BroadcastReceiver to handle scan result
    private val dataReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            if (intent?.action == ACTION) {
                val scannedData = intent.getStringExtra("data")
                val barcodeTypeStr = intent.getStringExtra("CODE_TYPE")

                scannedData?.let {
                    val barcodeType = getBarcodeType(barcodeTypeStr)
                    handleScannedData(it, barcodeType)
                }
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register the broadcast receiver
    private fun connectScanner() {
        try {
            val filter = IntentFilter(ACTION)
            registerReceiver(dataReceiver, filter)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister the broadcast receiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(dataReceiver)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Handle the scanned data and update the UI
    private fun handleScannedData(data: String, type: String) {
        Log.d("MainActivity", "Scanned Data: $data, Type: $type")
        runOnUiThread {
            textView.text = "Scanned Data: $data, Type: $type"
        }
    }

    // Map the barcode type based on the string value from the scan result
    private fun getBarcodeType(typeStr: String?): String {
        return when (typeStr?.toUpperCase()) {
            "EAN13" -> "EAN"
            "DATAMATRIX" -> "DATA_MATRIX"
            "PDF417" -> "PDF417"
            "EAN128" -> "GS1"
            "CODE128" -> "CODE128"
            else -> "UNKNOWN"
        }
    }
}
```
## Bluebird Integration

#### Broadcast Receiver for Bluebird

The **Bluebird** scanner broadcasts scan events using several **Intent actions**, including `"kr.co.bluebird.android.bbapi.action.BARCODE_CALLBACK_DECODING_DATA"`, which contains the scanned barcode data.

#### Key Elements:

-   **Actions**:
    -   `kr.co.bluebird.android.bbapi.action.BARCODE_CALLBACK_DECODING_DATA` (main action for barcode data)
    -   `kr.co.bluebird.android.bbapi.action.BARCODE_OPEN` (open the barcode scanner)
    -   `kr.co.bluebird.android.bbapi.action.BARCODE_SET_TRIGGER` (trigger scan)
    -   `kr.co.bluebird.android.bbapi.action.BARCODE_CLOSE` (close the barcode scanner)
    -   Additional actions for barcode settings and status callbacks
-   **Data Keys**:
    -   `EXTRA_BARCODE_DECODING_DATA` (the scanned barcode data as a byte array)

#### Code Implementation

Below is the implementation for handling scan data from **Bluebird** devices.

#### Kotlin Implementation:
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private lateinit var scanButton: Button
    private val ACTION_BARCODE_CALLBACK_DECODING_DATA = "kr.co.bluebird.android.bbapi.action.BARCODE_CALLBACK_DECODING_DATA"
    private val ACTION_BARCODE_OPEN = "kr.co.bluebird.android.bbapi.action.BARCODE_OPEN"
    private val ACTION_BARCODE_SET_TRIGGER = "kr.co.bluebird.android.bbapi.action.BARCODE_SET_TRIGGER"
    private val EXTRA_BARCODE_DECODING_DATA = "EXTRA_BARCODE_DECODING_DATA"
    private var barcodeHandle: Int = 0

    // BroadcastReceiver to handle scan result
    private val barcodeReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            if (intent?.action == ACTION_BARCODE_CALLBACK_DECODING_DATA) {
                val scannedData = intent.getByteArrayExtra(EXTRA_BARCODE_DECODING_DATA)
                scannedData?.let {
                    handleScannedData(String(it))
                }
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize TextView to display scanned data
        textView = findViewById(R.id.textView)
        
        // Initialize Button to trigger scanning
        scanButton = findViewById(R.id.scanButton)
        scanButton.setOnClickListener {
            startScan()
        }
    }

    override fun onResume() {
        super.onResume()
        connectScanner()
    }

    override fun onPause() {
        super.onPause()
        disconnectScanner()
    }

    // Connect to the scanner and register the broadcast receiver
    private fun connectScanner() {
        try {
            val filter = IntentFilter(ACTION_BARCODE_CALLBACK_DECODING_DATA)
            registerReceiver(barcodeReceiver, filter)

            val openIntent = Intent(ACTION_BARCODE_OPEN).apply {
                putExtra("EXTRA_HANDLE", barcodeHandle)
            }
            sendBroadcast(openIntent)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during connection: ${e.message}")
        }
    }

    // Disconnect the scanner and unregister the broadcast receiver
    private fun disconnectScanner() {
        try {
            unregisterReceiver(barcodeReceiver)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error during disconnect: ${e.message}")
        }
    }

    // Start scanning by sending a broadcast
    private fun startScan() {
        try {
            val scanIntent = Intent(ACTION_BARCODE_SET_TRIGGER).apply {
                putExtra("EXTRA_HANDLE", barcodeHandle)
                putExtra("EXTRA_INT_DATA2", 1) // Trigger ON
            }
            sendBroadcast(scanIntent)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error starting scan: ${e.message}")
        }
    }

    // Stop scanning by sending a broadcast
    private fun stopScan() {
        try {
            val stopIntent = Intent(ACTION_BARCODE_SET_TRIGGER).apply {
                putExtra("EXTRA_HANDLE", barcodeHandle)
                putExtra("EXTRA_INT_DATA2", 0) // Trigger OFF
            }
            sendBroadcast(stopIntent)
        } catch (e: Exception) {
            Log.e("MainActivity", "Error stopping scan: ${e.message}")
        }
    }

    // Handle the scanned data and update the UI
    private fun handleScannedData(data: String) {
        Log.d("MainActivity", "Scanned Data: $data")
        runOnUiThread {
            textView.text = "Scanned Data: $data"
        }
    }
}
```
