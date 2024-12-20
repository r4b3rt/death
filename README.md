# Death [![Build Status](https://travis-ci.org/vrecan/death.svg?branch=master)](https://travis-ci.org/vrecan/death) [![Coverage Status](https://coveralls.io/repos/github/vrecan/death/badge.svg?branch=master)](https://coveralls.io/github/vrecan/death?branch=master) [![Go Reference](https://pkg.go.dev/badge/github.com/vrecan/death/v3.svg)](https://pkg.go.dev/github.com/vrecan/death/v3) [![Join the chat at https://gitter.im/vrecan/death](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/vrecan/death?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<p>Simple library to make it easier to manage the death of your application.</p>

## Get The Library

Use gopkg.in to import death based on your logger.

| Version | Go Get URL                                                                           | source                                                           | doc                                                  | Notes                                                                                                                                                                                                                                                                                                |
| ------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3.x     | [github.com/vrecan/death/v3](https://github.com/vrecan/death/tree/release-branch.v3) | [source](https://github.com/vrecan/death/tree/release-branch.v3) | [doc](https://pkg.go.dev/github.com/vrecan/death/v3) | This removes the need for an independent logger. By default death will not log but will return an error if all the closers do not properly close. If you want to provide a logger just satisfy the deathlog.Logger interface. This also uses go modules so import it as `github.com/vrecan/death/v3` |
| 2.x     | [gopkg.in/vrecan/death.v2](https://gopkg.in/vrecan/death.v2)                         | [source](https://github.com/vrecan/death/tree/v2.0)              | [doc](https://godoc.org/gopkg.in/vrecan/death.v2)    | This supports loggers who _do not_ return an error from their `Error` and `Warn` functions like [logrus](https://github.com/sirupsen/logrus)                                                                                                                                                         |
| 1.x     | [gopkg.in/vrecan/death.v1](https://gopkg.in/vrecan/death.v1)                         | [souce](https://github.com/vrecan/death/tree/v1.0)               | [doc](https://godoc.org/gopkg.in/vrecan/death.v1)    | This supports loggers who _do_ return an error from their `Error` and `Warn` functions like [seelog](https://github.com/cihub/seelog)                                                                                                                                                                |

Example

```bash
go get github.com/vrecan/death/v3
```

## Use The Library

```go
package main

import (
	DEATH "github.com/vrecan/death/v3"
	SYS "syscall"
)

func main() {
	death := DEATH.NewDeath(SYS.SIGINT, SYS.SIGTERM) //pass the signals you want to end your application
	//when you want to block for shutdown signals
	death.WaitForDeath() // this will finish when a signal of your type is sent to your application
}
```

### Close Other Objects On Shutdown

<p>One simple feature of death is that it can also close other objects when shutdown starts</p>

```go
package main

import (
	"log"
	DEATH "github.com/vrecan/death/v3"
	SYS "syscall"
	"io"
)

func main() {
	death := DEATH.NewDeath(SYS.SIGINT, SYS.SIGTERM) //pass the signals you want to end your application
	objects := make([]io.Closer, 0)

	objects = append(objects, &NewType{}) // this will work as long as the type implements a Close method

	//when you want to block for shutdown signals
	err := death.WaitForDeath(objects...) // this will finish when a signal of your type is sent to your application
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}
}

type NewType struct {
}

func (c *NewType) Close() error {
	return nil
}

```

### Or close using an anonymous function

```go
package main

import (
	DEATH "github.com/vrecan/death/v3"
	SYS "syscall"
)

func main() {
	death := DEATH.NewDeath(SYS.SIGINT, SYS.SIGTERM) //pass the signals you want to end your application
	//when you want to block for shutdown signals
	death.WaitForDeathWithFunc(func(){
		//do whatever you want on death
	})
}
```

# Release Process

## Rules for release branches:

- If you are releasing a new major version you need to branch off of master into a branch `release-branch.v#` (example `release-branch.v2` for a 2.x release)
- If you are releasing a minor or patch update to an existing major release make sure to merge master into the release branch

## Rules for tagging and publishing the release

When you are ready to publish the release make sure you...

1. Merge your changes into the correct release branch.
2. Check out the release branch locally (example: `git pull origin release-branch.v3`)
3. Create a new tag for the specific release version you will publish (example: `git tag v3.0.1`)
4. Push the tag up to github (example: `git push origin v3.0.1`)
5. Go to the release tab in github
6. Select the target branch as the release branch and type in the tag name (tagname should include `v` so example: `v3.0.1`)
7. Write a title and a well worded description on exactly what is in this change
8. Click publish release
