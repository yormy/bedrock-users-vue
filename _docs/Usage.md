# Using components
```
import { HelloComponent } from "bedrock-users-vue";

components: {
    HelloComponent
}
  
```

# Using translations
in <lang>/_index
```
import { translations as bedrockUsersTranslations } from 'bedrock-users-vue';
...

  modules: {
    user: bedrockUsersTranslations,
  },
```


#include root components
app.js
```
 require("./../../vendor/mexion/multilingual-admin/resources/assets/package.js")
```
