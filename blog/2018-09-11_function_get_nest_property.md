# Simple function to get nest property of object in Javascript

This function is used to get nested property of an object through null checking and property exist checking. Here is the function:

```
const getNestedProperty = (obj, prop, defaultValue = "") => {

  for (var i = 0; i < prop.length; i++) {
    if (!obj) {
      return defaultValue;
    }
	if (!obj.hasOwnProperty(prop[i])) {
      return defaultValue;
    }
    obj = obj[prop[i]];
  }
  if(!obj) {
	return defaultValue;
  } else {
	return obj;
  }
}
```

Usage is like below:
For example we have an object like this:
```
var person = {
    name: {
        zh: "陳大文",
        en: "Chan Tai Man"
    },
    "age": 25,
    "contact": {
        "mobile": "12345678",
        "email": "chantaiman@gmail.com"
    }
};
```

And usage:
```
getNestedProperty(person, ["age"]); // Return 25
getNestedProperty(person, ["name", "zh"]); // Return "陳大文"
getNestedProperty(person, ["contact", "home"]); // Return "" since home is not exist in contact property, and default for defaultValue is ""
getNestedProperty(person, ["contact", "home"], "unknown"); // Return "unknown" since default is set by 3rd parameter
