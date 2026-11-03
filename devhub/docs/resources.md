# Resources and configuration

## Shared resources

- **api** (`httpConnResource`) in `OrderProcessing.module/Resources/orderprocessing/module/api.httpConnResource`
- **JMSConnectionResource** (`jmsConnResource`) in `OrderProcessing.module/Resources/orderprocessing/module/JMSConnectionResource.jmsConnResource`


## Configuration variables

Values from the application's default profile, falling back to the module defaults. Names that
look like secrets, and any value that is an obfuscated password or a `%%placeholder%%`, are left
out on purpose.

| Variable | Value |
|---|---|
| `LogLevel` | `INFO` |
| `EMSURL` | `tcp://192.168.1.30:9011` |
| `EMSUserName` | `admin` |

