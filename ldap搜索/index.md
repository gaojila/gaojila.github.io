# ldap搜索


# 前言

闲着没事，看 python 不爽，决定重构 mutt-lday.py

# 实现

- viper 读取配置文件
- ldap 解析配置文件

## 代码实现

```Go
// Package main provides ...
package main

import (
	"fmt"
	"os"

	log "github.com/sirupsen/logrus"
	"github.com/spf13/viper"
	"gopkg.in/ldap.v2"
)

type connection struct {
	Server string `json:"server"`
	Port   int    `json:"port"`
	Basedn string `json:"basedn"`
}

type auth struct {
	User     string `json:"user"`
	Password string `json:"password"`
}

type result struct {
	OptionalColumn string `json:"optionalcolumn"`
}

type searchConfig struct {
	Connection connection `json:"connection"`
	Auth       auth       `json:"auth"`
	Result     result     `json:"result"`
}

type searchResult struct {
	Mail string
	Name string
}

var matchAttributes = []string{"uid", "cn"}
var displayAttributes = []string{"mail", "cn"}

func (conf *searchConfig) init() {
	viper.SetConfigName("mutt-ldap")
	viper.SetConfigType("json")
	viper.AddConfigPath("./")
	viper.AddConfigPath("~/.config/")

	if err := viper.ReadInConfig(); err != nil {
		log.Fatalf("Error reading config file, %s", err)
	}

	if err := viper.Unmarshal(conf); err != nil {
		log.Fatalf("unable to decode into struct, %v", err)
	}

	log.Tracef("searchConfig.connection %v", conf.Connection)
	log.Tracef("searchConfig.auth %v", conf.Auth)
	log.Tracef("searchConfig.result %v", conf.Result)
}

func searchLdap(conf *searchConfig, term string) ([]searchResult, error) {

	filter := "(&(|"
	for _, attr := range matchAttributes {
		filter = fmt.Sprintf("%s(%s=*%s*)", filter, attr, term)
	}

	filter = fmt.Sprintf("%s)(mail=*))", filter)

	conn, err := ldap.Dial("tcp", fmt.Sprintf("%s:%d", conf.Connection.Server, conf.Connection.Port))
	if err != nil {
		log.Fatalf("Conn to server fail, %s", err)
	}

	defer conn.Close()

	if err := conn.Bind(conf.Auth.User, conf.Auth.Password); err != nil {
		log.Fatalf("Bind to server fail, %s", err)
	}
	searchresult := ldap.NewSearchRequest(conf.Connection.Basedn, ldap.ScopeWholeSubtree, ldap.NeverDerefAliases, 0, 0, false, filter, displayAttributes, nil)

	sr, err := conn.Search(searchresult)
	if err != nil {
		log.Fatalf("search error, %s", err)
	}

	res := make([]searchResult, len(sr.Entries))
	for idx, entry := range sr.Entries {
		res[idx].Mail = entry.GetAttributeValue("mail")
		res[idx].Name = entry.GetAttributeValue("cn")
	}
	return res, nil
}

func printResult(ldapRes []searchResult) {
	for _, entry := range ldapRes {
		fmt.Printf("%s\t%s\n", entry.Mail, entry.Name)
	}
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("usage:error")
		os.Exit(0)
	}

	var c searchConfig
	c.init()

	ldapRes, err := searchLdap(&c, os.Args[1])
	if err != nil {
		log.Fatal(err)
		os.Exit(1)
	}

	printResult(ldapRes)

}
```

## json 文件配置

```json
  ~/go/src/day_test                                                     15:08:27 
cat mutt-ldap.json
{
    "connection": {
        "server": "*********",
        "port": "389",
        "basedn": "DC=********,DC=com"
    },
    "auth": {
        "user": "*******",
        "password": "*********"
    },
    "result": {
        "optionalcolumn": "sAMAccountName"
    }
}
```

