# XRpage
## README

### Описание проекта

Этот проект демонстрирует пример использования WebXR с A-Frame для создания 360-градусного видео с поддержкой виртуальной реальности (VR). Пользователь может воспроизводить/пауза видео и входить в режим VR, если его устройство поддерживает эту функцию.

### Структура проекта

Проект включает следующие основные компоненты:

1. **HTML**: Основной файл разметки.
2. **CSS**: Стили для кнопок управления.
3. **JavaScript**: Логика управления видео и VR.

### Основные компоненты

#### HTML

```html
<!DOCTYPE html>
<html>

<head>
    <title>WebXR with A-Frame</title>
    <meta name="description" content="WebXR Example with A-Frame">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <link rel='stylesheet' href='VRpage.css'>
    <script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
</head>

<body>
    <a-scene xr-mode-ui="enterVRButton: #xrButton;">
        <header>
            <details open>
                <summary>360 Video</summary>
            </details>
            <button id="xrButton" disabled>VR NOT FOUND</button>
            <button id="playPauseButton">Play/Pause Video</button>
        </header>
        <a-assets>
            <video id="video" src="great-barrier-reef-preview.mp4" autoplay loop="true" crossorigin="anonymous"></video>
        </a-assets>

        <a-videosphere src="#video" rotation="0 -90 0"></a-videosphere>

        <a-entity camera look-controls>
            <a-cursor></a-cursor>
        </a-entity>
    </a-scene>

    <script src="main.js"></script>
</body>

</html>
```

Этот файл содержит основную структуру сцены A-Frame, включая видео сферу, камеру и пользовательский интерфейс с кнопками для управления видео и входа в VR режим.

#### CSS

Файл `VRpage.css` используется для стилей кнопок и их позиционирования.

```css
#playPauseButton {
    position: absolute;
    top: 10px;
    left: 10px;
    z-index: 1000;
    padding: 10px;
    background-color: white;
    border: 1px solid black;
    cursor: pointer;
}

#xrButton {
    position: absolute;
    top: 10px;
    right: 10px;
    z-index: 1000;
    padding: 10px;
    background-color: white;
    border: 1px solid black;
    cursor: pointer;
}
```

#### JavaScript

Файл `main.js` содержит логику для управления воспроизведением видео и проверки поддержки WebXR:

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const video = document.getElementById('video');
    const playPauseButton = document.getElementById('playPauseButton');
    const xrButton = document.getElementById('xrButton');

    playPauseButton.addEventListener('click', () => {
        if (video.paused) {
            video.play();
        } else {
            video.pause();
        }
    });

    if (navigator.xr) {
        console.log('WebXR API доступен');
        navigator.xr.isSessionSupported('immersive-vr').then((supported) => {
            if (supported) {
                console.log('VR поддерживается');
                xrButton.innerText = 'Enter VR';
                xrButton.disabled = false;

                xrButton.addEventListener('click', () => {
                    navigator.xr.requestSession('immersive-vr', {
                        requiredFeatures: ['local-floor']
                    }).then((session) => {
                        onSessionStarted(session);
                    });
                });
            } else {
                console.log('VR не поддерживается');
            }
        });
    } else {
        console.log('WebXR API не доступен');
    }

    function onSessionStarted(session) {
        session.addEventListener('end', onSessionEnded);

        let gl = document.createElement('canvas').getContext('webgl', { xrCompatible: true });
        let xrLayer = new XRWebGLLayer(session, gl);

        session.updateRenderState({ baseLayer: xrLayer });

        session.requestReferenceSpace('local').then((refSpace) => {
            function onXRFrame(t, frame) {
                let pose = frame.getViewerPose(refSpace);
                session.requestAnimationFrame(onXRFrame);
            }
            session.requestAnimationFrame(onXRFrame);
        });
    }

    function onSessionEnded(event) {
        video.pause();
    }
});
```

### В процессе

1. **Добавление кнопки выхода из VR**: Этот компонент также может включать кнопку для выхода из VR. В данном примере можно добавить эту функциональность, используя следующий метод:

```javascript
function createExitVRButton (onClick) {
    var exitButton; // здесь var, так как в xr-mode-ui.js в aframe используется также var
    var wrapper;

    wrapper = document.createElement('div');
    wrapper.classList.add('a-exit-vr');
    wrapper.setAttribute(constants.AFRAME_INJECTED, '');
    exitButton = document.createElement('button');
    exitButton.className = 'a-exit-vr-button';
    exitButton.setAttribute('title', 'Exit VR mode');
    exitButton.setAttribute(constants.AFRAME_INJECTED, '');
    if (utils.device.isMobile()) { applyStickyHoverFix(exitButton); }
    wrapper.appendChild(exitButton);
    exitButton.addEventListener('click', function (evt) {
        onClick();
        evt.stopPropagation();
    });
    return wrapper;
}

document.body.appendChild(createExitVRButton(() => {
    document.querySelector('a-scene').exitVR();
}));
```

Эта кнопка будет отображаться только когда пользователь находится в VR-режиме и позволит выйти из него при нажатии.
