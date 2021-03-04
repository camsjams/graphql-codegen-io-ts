# graphql-codegen-io-ts

## Usage
TBD

## Custom Scalars
The templates support a custom generator config for custom scalars that you need to define in your `gql-gen.json`:
```json
{
  "generatorConfig": {
    "customScalars": "src/io-types/scalars"
  }
}
```

It needs to export all the custom scalars used in your schema:
```typescript
import * as t from 'io-ts'
import moment from 'moment'

// represents a Date from an ISO string
export const Datetime = new t.Type<moment.Moment, string>(
  'Datetime',
  (mixed): mixed is moment.Moment => moment.isMoment(mixed),
  (mixed, context) =>
    t.string.validate(mixed, context).chain(str => {
      const instance = moment(str)
      return instance.isValid() ? t.success(instance) : t.failure(str, context)
    }),
  instance => instance.toISOString()
)

export type Datetime = moment.Moment
```

## Examples
Tranit decoding, i.e. for apollo-client:
```typescript
import * as t from 'io-ts'
import { Operations } from 'schema/__generated__/io-types'

type Op = { operationName: Operations.operationType }

export function decoder<O extends Op>(operation: O) {
  return ({ data, ...response }) => ({
      ...response,
      data: Operations[operation.operationName]
        .Operation
        .decode(data)
        .getOrElseL(errors => {
          let failures = failure(errors).join('\n');
          console.error(failures);
          throw new Error(failures)
        })
    })
}
```

