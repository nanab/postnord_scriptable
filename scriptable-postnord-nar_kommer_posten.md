# Scriptable Postnord Widget

<p align="center" >
    
</p>

- [Scriptable Postnord Widget](#scriptable-postnord-widget)
  - [Instruktioner](#instruktioner)
  - [Code](#code)

## Instruktioner


- Kopiera skriptet under [Code](#Code) sektionen til lett nytt skript i scriptable.
- Lägg till ett medium size Scriptable widget på hemskärmen.
- Långtryck på widgeten och välj "Edit Widget".
- Set the `Script` to be the script you just created and `When Interacting` to `Run Script` which will then launch Calendar app when you tap on the widget.


---

- Ändra variabeln postCode till ditt postnummer.


## Code

_You are free to use and modify the code however you wish. There are likely going to be bugs in this code as I have not tested it for every edge case._

```js
// Postkod för din postutdelning
const postCode = "37295";


let req = new Request("https://portal.postnord.com/api/sendoutarrival/closest?postalCode=" + postCode)
req.headers = {
  "Content-Type": "application/json"
}
req.method = "GET";
let json = await req.loadJSON()

let postalCode = json["postalCode"];
let city = json["city"];
let delivery = json["delivery"];
let upcoming = json["upcoming"];

async function createWidget() {
  // Create new empty ListWidget instance
  let lw = new ListWidget();

  // Set new background color
  lw.backgroundColor = new Color("#000000"); //svart bakgrund

  let headStack = lw.addText("När kommer posten?");
  headStack.centerAlignText();
  lw.addSpacer(10);
  
  // Skapadatum och sätt tid för nästa uppdatering, tidigast 30 min in på nästa dag.
  var d = new Date();
  datenowday = d.getDate().toString(); //dagensdatum
  d.setDate(d.getDate() + 1);
  dateTomorrow = d.getDate().toString(); //datum imorgon
  d.setHours(0);
  d.setMinutes(30);
  lw.refreshAfterDate = d;

  //läs ut datum delivery
  let deliveryDate = delivery.slice(0, 2);
  deliveryDate = deliveryDate.trim();
  let deliveryText;
  let deliveryColor;
  if (deliveryDate == datenowday) {
   deliveryText = "Idag";
   deliveryColor = "#35de3b"; // Grön färg
  } else if (deliveryDate == dateTomorrow){
    deliveryText = "Imorgon";
    deliveryColor = "#de4035"; //Röd färg
  } else {
    deliveryText = "Kommer den " + deliveryDate + ":e";
    deliveryColor = "#de4035"; //Röd färg
  }
  let deliveryStack = lw.addText(deliveryText);
  deliveryStack.textColor = new Color(deliveryColor)
  deliveryStack.centerAlignText();
  lw.addSpacer(15)
  
  let upcomingStack = lw.addText(upcoming);
  upcomingStack.centerAlignText();
  upcomingStack.font = Font.lightSystemFont(10);

  lw.addSpacer(30)

  // När uppdaterades widgeten senast
  d = new Date()
  let hour = d.getHours();

  if (hour < 10) hour = "0" + hour;
  let min = d.getMinutes();
  if (min < 10) min = "0" + min;

  let time = lw.addText("Uppdaterad: " + hour + ":" + min);
  time.centerAlignText();
  time.font = Font.lightSystemFont(8);

  // Return the created widget
  return lw;
}

let widget = await createWidget();

// Check where the script is running
if (config.runsInWidget) {
  Script.setWidget(widget);
} else {
  // Show the medium widget inside the app
  widget.presentMedium();
}
Script.complete();