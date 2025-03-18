# A Modular AR Tarot App.

Project Overview: Interactive Model Display Platform Based on Web NFC and WebAR

This project is a lightweight, user-friendly mobile Web application that combines Web NFC and WebAR technologies. Using the phone’s built-in NFC sensor, it reads physical NFC tags and loads associated 3D models based on the tag’s information, enabling augmented reality (AR) display. The project uses P5.js as the front-end framework, A-Frame for building AR scenes, and a local development environment powered by Termux virtual machine and Python HTTP server for model resource management. The goal of this project is to explore the potential applications of NFC and AR technologies in architecture, cultural promotion, education, and creative exhibitions.

Tech Stack:

Front-End Framework: P5.js

AR Engine: A-Frame

NFC Data Reading: Web NFC API

3D Model Format: glTF (.glb)

Local Server: Termux Virtual Machine + Python HTTP Server


Workflow:

Open the P5.js-based Web application on a mobile device.

The user clicks the “Scan NFC” button and scans a physical NFC tag via the Web NFC API.

The system retrieves the corresponding 3D model from a model mapping table based on the tag’s information.

The model is loaded into an A-Frame AR scene and displayed in real-time.


Application Scenarios:

Architectural Design Display: Quickly switch between different architectural models.

Cultural and Historical Promotion: Use physical NFC-tagged objects to showcase virtual cultural artifacts.

Education and Training: Visualize abstract concepts through 3D models to enhance learning experiences.

Creative Interactive Experiences: Combine NFT and WebAR to create personalized narratives and interactions.


Current Progress and Next Steps:

The application has successfully implemented NFC tag reading and dynamic 3D model loading, with a fully functional local development environment. The next steps include expanding the model library, enhancing model interactivity, and exploring potential integration with open-source AR platforms such as Meta Spark AR.

```
code frontend (Final)(Updated 250318)

<!DOCTYPE html>
<html>
<head>
  <title>Modular Tarot AR</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.min.js"></script>
  <script src="https://aframe.io/releases/1.4.0/aframe.min.js"></script>
  <script>
    // NFC tag to model mapping
    const modelMap = {
      "13:98:cb:41": { A: "models/frame1.glb", B: "models/content1.glb", C: "models/decor1.glb" },
      "e3:26:e1:42": { A: "models/frame2.glb", B: "models/content2.glb", C: "models/decor2.glb" },
      "13:b6:13:43": { A: "models/frame3.glb", B: "models/content3.glb", C: "models/decor3.glb" },
      "13:3b:df:41": { A: "models/frame4.glb", B: "models/content4.glb", C: "models/decor4.glb" },
      "b3:8c:40:42": { A: "models/frame5.glb", B: "models/content5.glb", C: "models/decor5.glb" },
      "83:dd:72:43": { A: "models/frame6.glb", B: "models/content6.glb", C: "models/decor6.glb" }
    };

    let scanCount = 0; // Total scan count (accumulative across tags)
    const moduleTypes = ["A", "B", "C"];

    const loadPositions = {
      A: "0 1 0",
      B: "0 1.1 0",
      C: "0 1.2 0"
    };

    async function readNFC() {
      if ("NDEFReader" in window) {
        try {
          const ndef = new NDEFReader();
          await ndef.scan();
          ndef.onreading = event => {
            const tagId = event.serialNumber;
            console.log("NFC Tag ID:", tagId);

            if (!modelMap[tagId]) {
              console.log("Unrecognized NFC tag!");
              return;
            }

            // Reset scene and scan count every 4 scans
            if (scanCount >= 3) {
              resetScene();
              scanCount = 0;
            }

            // Load the next model (A -> B -> C)
            loadNextModule(tagId);
            scanCount++;
          };
        } catch (error) {
          console.error("NFC scanning error:", error);
        }
      } else {
        alert("Web NFC is not supported on this device!");
      }
    }

    function loadNextModule(tagId) {
      const modelData = modelMap[tagId];
      const moduleType = moduleTypes[scanCount];

      if (modelData[moduleType]) {
        loadModule(moduleType, modelData[moduleType]);
      }
    }

    function loadModule(moduleType, modelPath) {
      let container = document.querySelector("#tarotContainer");

      // Check if the model type already exists to avoid duplicates
      let existingModule = document.getElementById(moduleType);

      if (existingModule) {
        existingModule.setAttribute("gltf-model", modelPath); // Replace existing model
      } else {
        let moduleEntity = document.createElement("a-entity");
        moduleEntity.setAttribute("id", moduleType);
        moduleEntity.setAttribute("gltf-model", modelPath);
        moduleEntity.setAttribute("position", loadPositions[moduleType]);
        moduleEntity.setAttribute("scale", "0.5 0.5 0.5");
        container.appendChild(moduleEntity);
      }

      console.log(`${moduleType} model loaded: ${modelPath}`);
    }

    function resetScene() {
      const container = document.querySelector("#tarotContainer");
      container.innerHTML = ""; // Clear the scene
      console.log("Scene reset");
    }
  </script>
</head>
<body>
  <button onclick="readNFC()">Scan NFC</button>
  <a-scene embedded>
    <a-camera position="0 1.6 0"></a-camera>
    <a-entity id="tarotContainer" position="0 0 -3"></a-entity>
  </a-scene>
</body>
</html>

```

```

code frontend (Old)

let modelMap = {
  "13:98:cb:41": "model1.glb",
  "tag2": "model2.glb",
  "tag3": "model3.glb"
};

function setup() {
  createCanvas(400, 200);
  background(200);
  textSize(16);
  textAlign(CENTER, CENTER);
  text("Tap NFC tag to load AR model", width / 2, height / 2);

  let scanButton = createButton('Scan NFC');
  scanButton.position(150, 150);
  scanButton.mousePressed(readNFC);

  let scene = createDiv('');
  scene.id('scene');
  scene.html('<a-scene embedded><a-camera position="0 1.6 0"></a-camera><a-entity id="arModel" position="0 0 -2"></a-entity></a-scene>', true);
}

async function readNFC() {
  if ('NDEFReader' in window) {
    try {
      const ndef = new NDEFReader();
      await ndef.scan();
      ndef.onreading = event => {
        const tagId = event.serialNumber;
        console.log('NFC Tag ID:', tagId);
        loadARModel(modelMap[tagId] || 'default.glb');
      };
    } catch (error) {
      console.error('NFC Error:', error);
    }
  } else {
    alert('Web NFC not supported on this device.');
  }
}

function loadARModel(modelPath) {
  let modelEntity = select('#arModel');
  if (modelEntity) {
    modelEntity.attribute('gltf-model', modelPath);
  }
}

```

```

code Backend (Termux Simulated Python HTTP Server)

$ pkg update
$ pkg install python
$ termux-setup-storage
$ cd /storage/emulated/0/XXXXX
$ python -m http.server XXXX

```


## Subsection

1. one
2. two

* bullets
* bullets

## Reference

https://developer.mozilla.org/zh-CN/docs/Web/API/Web_NFC_API
https://w3c.github.io/web-nfc/
https://aframe.io/blog/
https://developer.chrome.com/docs/capabilities/nfc?hl=zh-cn&utm_source
https://developer.mozilla.org/zh-CN/docs/Web/API/NDEFReader
https://wenku.csdn.net/answer/03a526a2544446f3afe58aad7eb7369f?utm_source
https://aframe.io/blog/arjs/

```
const ndef = new NDEFReader();

function read() {
  return new Promise((resolve, reject) => {
    const ctlr = new AbortController();
    ctlr.signal.onabort = reject;
    ndef.addEventListener("reading", event => {
      ctlr.abort();
      resolve(event);
    }, { once: true });
    ndef.scan({ signal: ctlr.signal }).catch(err => reject(err));
  });
}

read().then(({ serialNumber }) => {
  console.log(serialNumber);
});
```

```
const ndef = new NDEFReader();
await ndef.scan();
ndef.onreading = (event) => {
  const externalRecord = event.message.records.find(
    record => record.type == "example.com:smart-poster"
  );

  let action, text;

  for (const record of externalRecord.toRecords()) {
    if (record.recordType == "text") {
      const decoder = new TextDecoder(record.encoding);
      text = decoder.decode(record.data);
    } else if (record.recordType == ":act") {
      action = record.data.getUint8(0);
    }
  }

  switch (action) {
    case 0: // do the action
      console.log(`Post "${text}" to timeline`);
      break;
    case 1: // save for later
      console.log(`Save "${text}" as a draft`);
      break;
    case 2: // open for editing
      console.log(`Show editable post with "${text}"`);
      break;
  }
};
```

```
const ndef = new NDEFReader();
ndef.scan().then(() => {
  console.log("Scan started successfully.");
  ndef.onreadingerror = () => {
    console.log("Cannot read data from the NFC tag. Try another one?");
  };
  ndef.onreading = event => {
    console.log("NDEF message read.");
  };
}).catch(error => {
  console.log(`Error! Scan failed to start: ${error}.`);
});
```

```
ndef.onreading = event => {
  const message = event.message;
  for (const record of message.records) {
    console.log("Record type:  " + record.recordType);
    console.log("MIME type:    " + record.mediaType);
    console.log("Record id:    " + record.id);
    switch (record.recordType) {
      case "text":
        // TODO: Read text record with record data, lang, and encoding.
        break;
      case "url":
        // TODO: Read URL record with record data.
        break;
      default:
        // TODO: Handle other records with record data.
    }
  }
};
```

```
const ndef = new NDEFReader();
let ignoreRead = false;

ndef.onreading = (event) => {
  if (ignoreRead) {
    return; // 待写入，忽略读取。
  }

  console.log("我们读取了一个标签，但在待写入期间没有读取！");
};

function write(data) {
  ignoreRead = true;
  return new Promise((resolve, reject) => {
    ndef.addEventListener(
      "reading",
      (event) => {
        // 检查是否要写入该标签，或拒绝写入。
        ndef
          .write(data)
          .then(resolve, reject)
          .finally(() => (ignoreRead = false));
      },
      { once: true },
    );
  });
}

await ndef.scan();
try {
  await write("你好，世界");
  console.log("我们已将数据写入标签！");
} catch (err) {
  console.error("出了一些问题", err);
}
```

```
<!DOCTYPE html>
<html>
<head>
    <title>Web NFC API Example</title>
</head>
<body>
    <h1>NFC Tag Reader</h1>
    <p>Place an NFC tag near your device to read its contents.</p>
    <div id="output"></div>
    <script src="nfc.js"></script>
</body>
</html>
```

```
// Check if the browser supports Web NFC API
if ('NDEFReader' in window) {
    // Create a new NDEFReader instance
    const reader = new NDEFReader();
    const output = document.getElementById('output');

    // Event listener for reading NFC tags
    reader.addEventListener('reading', ({ message, serialNumber }) => {
        output.textContent = `Serial Number: ${serialNumber}\n`;
        message.records.forEach((record) => {
            output.textContent += `Record Type: ${record.recordType}\n`;
            const textDecoder = new TextDecoder(record.encoding);
            output.textContent += `Data: ${textDecoder.decode(record.data)}\n`;
        });
    });

    // Start scanning for NFC tags
    reader.scan().then(() => {
        console.log('NFC scan started');
    }).catch((error) => {
        console.error(`Error: ${error}`);
    });
} else {
    console.error('Web NFC API not supported');
}
```

```
<!-- include A-Frame obviously -->
<script src="https://aframe.io/releases/0.6.0/aframe.min.js"></script>
<!-- include ar.js for A-Frame -->
<script src="https://jeromeetienne.github.io/AR.js/aframe/build/aframe-ar.js"></script>
<body style='margin : 0px; overflow: hidden;'>
  <a-scene embedded arjs>
    <!-- create your content here. just a box for now -->
    <a-box position='0 0.5 0' material='opacity: 0.5;'></a-box>
    <!-- define a camera which will move according to the marker position -->
    <a-marker-camera preset='hiro'></a-marker-camera>
  </a-scene>
</body>
```
