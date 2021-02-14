

# Building & Publishing

## step 1 build the package
```
npm install
npm run build
```

## step 2: Publish
### setup for npm
``` 
npm adduser
```

### publish
```
npm publish
```


# Typescript
``` 
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { SampleComponent } from "vue-component-library";

@Component({
  components: {
    SampleComponent
  }
})
export default class App extends Vue {}
</script>
```
