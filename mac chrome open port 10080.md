# mac chrome open port 10080

```sh
cd "/Applications/Google Chrome.app/Contents/MacOS/"
mv Google.real "Google Chrome"
mv "Google Chrome" Google.real
printf '#!/bin/bash\ncd "/Applications/Google Chrome.app/Contents/MacOS"\n"/Applications/Google Chrome.app/Contents/MacOS/Google.real" --explicitly-allowed-ports=10080,6000 "$@"\n' > Google\ Chrome
chmod u+x "Google Chrome"
```
