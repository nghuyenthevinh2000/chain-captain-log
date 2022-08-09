# The problem of go module

Go module mistakes github.com/cosmos/ibc-go/v5 revision v5.0.0-beta1 for revision v5.0.0

```
go.mod

module github.com/evmos/evmos/v7

go 1.18

require (
	github.com/cosmos/ibc-go/v5 v5.0.0-beta1
)
```

```
go.sum

github.com/cosmos/ibc-go/v5 v5.0.0-beta1
```

Why god has made go module understood this way?

There is no mention of v5.0.0 any where. Only "github.com/cosmos/ibc-go/v5 v5.0.0-beta1".

# The path of debugging

1. Checking go proxy to see all relevant tags of "github.com/cosmos/ibc-go/v5".

```
curl https://proxy.golang.org/github.com/cosmos/ibc-go/v5/@v/list

v5.0.0-beta1
```

So, go proxy correctly fetches all tags for ibc-go/v5. The problem could only be with how go module inteprets revision.

2. Could it be the order or semantic versioning somehow? pre - release could be automatically intepreted as main release somehow?

* https://go.dev/ref/mod#versions
* https://go.dev/ref/mod#glos-pre-release-version
* try "go get github.com/cosmos/ibc-go/v5@v5.0.0-beta1" on a different go workspace works fine. What could be the problem here? Maybe, package conflicts?

3. Wild guest: could it be stored version of ibc-go/v5 on directory $HOME/go/pkg/mod/github.com/cosmos/ibc-go and $HOME/go/pkg/mod/cache leads to problem?

Maybe somehow go module will get from cache instead of downloading which could lead to the problem

* delete two directories above, get again with no success.

4. go.mod and go.sum determines the revision that will be get. Maybe a fresh start could help.

* delete all occurrences of "github.com/cosmos/ibc-go/v5 v5.0.0-beta1" in go.mod, go.sum
* delete cache, delete saved version of ibc-go
* get again with no success

5. By this point, it could very highly be package conflicts.

* A package that uses mismatch version of ibc-go. This package is github.com/evmos/ethermint. It is the only package that also uses ibc-go.
* If package A is imported, package A uses ibc-go/v2@v2.0.0, importer B uses ibc-go/v5@v5.0.0-beta1. It seems that to make A and B compatible, go module has updated package A version of ibc-go to ibc-go/v5@v5.0.0. It updates based on semantic version, not on factual tag.

# The solution

In reality, here is the situation

```
go.mod

module github.com/evmos/evmos/v7

go 1.18

require (
	github.com/cosmos/ibc-go/v5 v5.0.0-beta1
)

replace (
	github.com/evmos/ethermint => github.com/yihuang/ethermint v0.6.1-0.20220720060948-0da1b21ba35d
)
```

1. The go.mod file is virtually wrong because it needs "github.com/evmos/ethermint" in "require". 
2. When "go mod tidy", it has prompted go mod to download "github.com/evmos/ethermint v0.0.0-00010101000000-000000000000".
3. Because "github.com/evmos/ethermint v0.0.0-00010101000000-000000000000" use ibc-go(v1), it has led to package conflict with github.com/evmos/evmos/v7.
4. This results in "go mod tidy" downloading "github.com/cosmos/ibc-go"(v1) at revision v5.0.0 (the semantic upgrade for compatibility) which is non-existance.
5. It also explains why "go get github.com/cosmos/ibc-go/v5" also downloads "github.com/cosmos/ibc-go v1.5.0" which is so strange. The download is for ethermint v0.0.0.
6. Add "github.com/evmos/ethermint v0.6.1-0.20220809055228-e70d8fcb562a" to "require" solves this problem cleanly because this version of ethermint also has ibc-go/v5.
7. "go mod tidy" works fine.

Final fix:

```
go.mod

module github.com/evmos/evmos/v7

go 1.18

require (
	github.com/cosmos/ibc-go/v5 v5.0.0-beta1
	github.com/evmos/ethermint v0.6.1-0.20220809055228-e70d8fcb562a
)
```

Should have added base version of github.com/evmos/ethermint to "require".
