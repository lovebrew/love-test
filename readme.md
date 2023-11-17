# Lövetest
Test suite for the [Löve](https://github.com/love2d/love) APIs, based off of [this issue](https://github.com/love2d/love/issues/1745).

Currently written for [Löve 12](https://github.com/love2d/love/tree/12.0-development), which is still in development.

---

## Features
- [x] Simple pass/fail tests in Lua with minimal setup 
- [x] Ability to run all tests with a simple command
- [x] Ability to see how many tests are passing/failing
- [x] Ability to run a subset of tests
- [x] Ability to easily run an individual test
- [x] Ability to see all visual results at a glance
- [x] Compare graphics test output with an expected output
- [x] Automatic testing that happens after every commit
- [x] No platform-specific dependencies / scripts

---

## Coverage
This is the status of all module tests currently.  
See the **Todo** section for outstanding tests that need writing.
| Module            | Done | Todo | Skip |
| ----------------- | ---- | ---- | ---- |
| 🟢 audio          |  28  |   0  |   0  |
| 🟢 data           |  12  |   0  |   0  |
| 🟢 event          |   4  |   0  |   2  |
| 🟢 filesystem     |  29  |   0  |   2  |
| 🟢 font           |   7  |   0  |   0  |
| 🟡 graphics       | 102  |   2  |   1  |
| 🟢 image          |   5  |   0  |   0  |
| 🟢 math           |  20  |   0  |   0  |
| 🟡 physics        |  22  |   4  |   0  |
| 🟢 sound          |   4  |   0  |   0  |
| 🟢 system         |   6  |   0  |   2  |
| 🟢 thread         |   5  |   0  |   0  |
| 🟢 timer          |   6  |   0  |   0  |
| 🟢 video          |   2  |   0  |   0  |
| 🟢 window         |  34  |   0  |   2  |

> The following modules are not covered as we can't really emulate input nicely:  
> `joystick`, `keyboard`, `mouse`, and `touch`

---

## Running Tests
The testsuite aims to keep things as simple as possible, and just runs all the tests inside Löve to match how they'd be used by developers in-engine.
To run the tests, download the repo and then run the main.lua as you would a Löve game, i.e:

WINDOWS: `& 'c:\Program Files\LOVE\love.exe' PATH_TO_TESTING_FOLDER --console`  
MACOS: `/Applications/love.app/Contents/MacOS/love PATH_TO_TESTING_FOLDER`  
LINUX: `./love.AppImage PATH_TO_TESTING_FOLDER`

By default all tests will be run for all modules.  
If you want to specify a module/s you can use:  
`--runSpecificModules filesystem,audio`  
If you want to specify only 1 specific method only you can use:  
`--runSpecificMethod filesystem write`

All results will be printed in the console per method as PASS, FAIL, or SKIP with total assertions met on a module level and overall level.  

When finished, the following files will be generated in the `/output` directory with a summary of the test results:
- an `XML` file in the style of [JUnit XML](https://www.ibm.com/docs/en/developer-for-zos/14.1?topic=formats-junit-xml-format)
- a `HTML` file that shows the report + any visual test results
- a `Markdown` file you can use with [this github action](https://github.com/ellraiser/love-test-report)
> An example of all types of output can be found in the `/examples`  
> The visual results of any graphic tests can be found in `/output/actual`

---

## Architecture
Each method and object has it's own test method written in `/tests` under the matching module name.

When you run the tests, a single TestSuite object is created which handles the progress + totals for all the tests.  
Each module has a TestModule object created, and each test method has a TestMethod object created which keeps track of assertions for that method. You can currently do the following assertions:
- **assertNotNil**(value)
- **assertEquals**(expected, actual, label)
- **assertNotEquals**(expected, actual, label)
- **assertRange**(actual, min, max, label)
- **assertMatch**({option1, option2, option3 ...}, actual, label) 
- **assertGreaterEqual**(expected, actual, label)
- **assertLessEqual**(expected, actual, label)
- **assertObject**(table)
- **assertPixels**(imgdata, pixeltable, label)
- **assertCoords**(expected, actual, label)

Example test method:
```lua
-- love.filesystem.read test method
-- all methods should be put under love.test.MODULE.METHOD, matching the API
love.test.filesystem.read = function(test)
  -- setup any data needed then run any asserts using the passed test object
  local content, size = love.filesystem.read('resources/test.txt')
  test:assertNotNil(content)
  test:assertEquals('helloworld', content, 'check content match')
  test:assertEquals(10, size, 'check size match')
  content, size = love.filesystem.read('resources/test.txt', 5)
  test:assertNotNil(content)
  test:assertEquals('hello', content, 'check content match')
  test:assertEquals(5, size, 'check size match')
  -- no need to return anything or cleanup, GCC is called after each method
end
```

Each test is run inside it's own coroutine - you can use `test:waitFrames(frames)` to pause the test for a small period if you need to check things that won't happen for a few seconds.

After each test method is ran, the assertions are totalled up, printed, and we move onto the next method! Once all methods in the suite are run a total pass/fail/skip is given for that module and we move onto the next module (if any)

For sanity-checking, if it's currently not covered or it's not possible to test the method we can set the test to be skipped with `test:skipTest(reason)` - this way we still see the method listed in the test output without it affected the pass/fail totals

---

## Todo 
Test classes that still need to be written:
- [ ] graphics.Mesh
- [ ] graphics.ParticleSystem
- [ ] physics.Body
- [ ] physics.Contact
- [ ] physics.Shape (this will include physics.Fixture properties)

---

## Runner Exceptions
The automated tests through Github work for the most part however there are a few exceptions that have to be accounted for due to limitations of the VMs and the graphics emulation used.  
These exceptions are either skipped, or handled by using a 1px or 1/255rgba tolerance - when run locally on real hardware, these tests pass fine at the default 0 tolerance.
- love.graphics.points - on MacOSX runners, points are offset by 1,1 when drawn
- love.graphics.setWireframe - on MacOSX runners, wireframes are offset by 1,1 when drawn
- love.graphica.arc - on MacOSX runners, some open fill arcs are drawn differently at low scale
- love.audio.RecordingDevice - these can't be emulated on runners

---

## Future
- [ ] add BMfont alts for font class tests (Rasterizer + GlyphData)
- [ ] graphics.isCompressed() should have an example of all compressed files
- [ ] ability to test loading different combinations of modules
- [ ] performance tests?
