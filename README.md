# Fast async array operations

via this [tweet](https://twitter.com/theKashey/status/1465510988942221313). `lfi` makes things fast


```js
import pMap from 'p-map'
import pFilter from 'p-filter'
import {
  pipe,
  asConcur,
  mapConcur,
  filterConcur,
  collectConcur,
  toArray,
} from 'lfi'

const delay = timeout => new Promise(resolve => setTimeout(resolve, timeout))
const mapDelays = [10, 1, 1]
const filterDelays = [1, 1, 10]

const array = [0, 1, 2]

async function doIt() {
  console.log('start')
  /* Takes 20 seconds! */
  const finalArray = await pFilter(
    await pMap(array, async index => {
      await delay(mapDelays[index] * 1000)
      return index
    }),
    async index => {
      await delay(filterDelays[index] * 1000)
      return true
    },
  )
  console.log('finalArray', finalArray)

  /* Takes 11 seconds! */
  const otherFinalArray = await pipe(
    asConcur(array),
    mapConcur(async index => {
      console.log('mapConcur', index)
      await delay(mapDelays[index] * 1000)
      console.log('Map done', index)
      return index
    }),
    filterConcur(async index => {
      console.log('filterConcur', index)
      await delay(filterDelays[index] * 1000)
      return false
    }),
    collectConcur(toArray),
  )
  console.log('otherFinalArray', otherFinalArray)
}

doIt()
```