---
title: Ultimate ComboBox in QML
date: 2017-11-22T23:28:25+00:00
author: "Taras Kushnir"
permalink: /ultimate-combobox-in-qml/
categories:
  - Programming
  - Qt
keywords:
  - combobox
  - custom
  - qml
  - qt
  - quick
  - ui
---
Everybody who wanted to customize UI of `ComboBox` in QML knows that it is only possible though crutches and hacks. Usually that's not a problem and people start implementing their own custom ComboBoxes that are docked to somewhere. You can see an example of such implementation below (the code is simplified). These sort of implementations have few big problems which I will cover afterwards.

```
Item {
    id: comboBox
    property alias model: dropDownItems.model

    signal comboIndexChanged(int index);

    Rectangle {
        id: header
        anchors.fill: parent
        color: "gray"

        Text { text: dropDownItems.currentItem.itemText }

        MouseArea {
            anchors.fill: parent
            onClicked: { comboBox.state = comboBox.state === "down" ? "" : "down" }
        }
    }

    Rectangle {
        id: dropDown
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.top: header.bottom
        visible: false
        height: 0
        onActiveFocusChanged: { if (!activeFocus) { comboBox.state = ""; } }

        ListView {
            id: dropDownItems
            anchors.fill: parent

            delegate: Rectangle {
                property alias itemText: modelData
                height: 20
                width: parent.width

                Text { text: modelData }

                MouseArea {
                    anchors.fill: parent
                    onClicked: {
                        comboBox.state = ""
                        if (index !== comboBox.selectedIndex) {
                            comboBox.selectedIndex = index
                            comboIndexChanged(index)
                        }
                    }
                }
            }
        }
    }

    states: State {
        name: "down";
        PropertyChanges {
            target: dropDown
            height: 20 * dropDownItems.count
            visible: true
        }
    }
}
```

This approach has 2 big flaws.

The very first problem with this approach is `z-index` management. You have to make sure that owner of your custom `ComboBox` is on top of everything possible so I set `z: 100500` every time I instantiated my object. But what if you have several instances which can overlap? Correct, you have to assign `z-index` dynamically every time..

The other problem with this way is the focus management in QML apps which I personally find quite clumsy and raw (but of course it could be just my lack of skill). Simple enough, you want to click outside to close the dropdown and .. usually that's just not possible without hacks. I've done horrible things to workaround this. I used to put huge `MouseArea` elements everywhere to emit `closeComboBoxes()` signal when it get's clicked and all the `ComboBox` in the area were subscribed to that signal.

So is there anything better?

<!--more-->

What I did is that I created 2 controls `ComboBoxHeader` and `ComboBoxDropdown`. The first one was the control I used to put in the QML here and there where I needed the `ComboBox` functionality. It's code is pretty simple:

```
/* ComboBoxHeader.qml */

Item {
    id: comboBox

    property var globalParent
    property var model
    property int selectedIndex

    signal comboItemSelected(int index);

    function openPopup() {
        /*THE MAGIC GOES HERE*/
    }

    Rectangle {
        id: header
        anchors.fill: parent
        color: "gray"

        Text { text: comboBox.model[comboBox.selectedIndex] }

        MouseArea {
            anchors.fill: parent
            onClicked: { openPopup() }
        }
    }
}
```

The important part there is `property var globalParent`. You should set this to some top-most `Item`-derived component which will serve as the _invisible background_. The dropdown is instantiated in relative to this root component with the following code:

```javascript
function openPopup() {
    // MAGIC:
    var marginPoint = comboBox.mapToItem(globalParent, 0, comboBox.height)

    var options = {
        model: comboBox.model,
        selectedIndex: comboBox.selectedIndex,
        leftPadding: marginPoint.x,
        topPadding: marginPoint.y,
    }

     var component = Qt.createComponent("ComboBoxDropdown.qml")
     var instance = component.createObject(globalParent, options)        
     instance.comboItemSelected.connect(comboBox.comboItemSelected)
}
```

And the other control (in file _ComboBoxDropdown.qml_) is a dropdown control with an `Item` root element (`globalParent`) filling everything on background. This root element intercepts all the mouse activity outside and correctly closes the popup if clicked.

```
/* ComboBoxDropdown.qml */

Item {
    id: dropdownComponent
    anchors.fill: parent

    property alias model: dropDownItems.model
    property alias selectedIndex: dropDownItems.currentIndex

    property double topPadding: 0
    property double leftPadding: 0

    signal comboItemSelected(int index)
    function closePopup() { dropdownComponent.destroy() }

    MouseArea {
        anchors.fill: parent
        onWheel: wheel.accepted = true
        onClicked: {
            mouse.accepted = true
            closePopup()
        }
    }

    Rectangle {
        id: dropDown
        anchors.top: parent.top
        anchors.left: parent.left
        anchors.leftMargin: dropdownComponent.leftPadding
        anchors.topMargin: dropdownComponent.topPadding
        color: "gray"
        visible: false
        width: 200
        height: 0
        state: "down"

        ListView {
            id: dropDownItems
            anchors.fill: parent

            delegate: Rectangle {
                width: parent.width
                height: 20

                Text { text: modelData }

                MouseArea {
                    anchors.fill: parent
                    onClicked: {
                        comboBox.state = ""
                        if (index !== comboBox.selectedIndex) {
                            comboBox.selectedIndex = index
                            comboItemSelected(index)
                        }
                    }
                }
            }
        }

        states: State {
            name: "down";
            PropertyChanges {
                target: dropDown;
                height: 20 * dropDownItems.count
                visible: true
            }
        }
    }
}
```

Now to use it you just go

```
ComboBoxHeader {
    globalParent: someTopMostItem
    model: ["Some", "data", "for", "the", "test", "here"]
    height: 20
    width: 200

    onComboItemSelected: { console.log(index) }
}
```

With this you can guarantee natural behavior of `ComboBox` and it is absolutely custom: you can style it as you want. So far this is the best option of customizable `ComboBox` I've seen anywhere.
