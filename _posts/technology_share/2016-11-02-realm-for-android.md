---
layout: post
tags: Realm
---


## Introduction


## Gifs

<img src="http://offfjcibp.bkt.clouddn.com/realmdemo.gif" width="30%" />

## Prerequisites


## Installation



```java
buildscript {

    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'
        classpath "io.realm:realm-gradle-plugin:2.1.1"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }

}
```



## Configuration


```java
RealmConfiguration config = new RealmConfiguration.Builder().build();
```


```java
// The RealmConfiguration is created using the builder pattern.
// The Realm file will be located in Context.getFilesDir() with name "myrealm.realm"
RealmConfiguration config = new RealmConfiguration.Builder()
  .name("myrealm.realm")
  .encryptionKey(getKey())
  .schemaVersion(42)
  .modules(new MySchemaModule())
  .migration(new MyMigration())
  .build();
// Use the config
Realm realm = Realm.getInstance(config);
```


```java
RealmConfiguration myConfig = new RealmConfiguration.Builder()
  .name("myrealm.realm")
  .schemaVersion(2)
  .modules(new MyCustomSchema())
  .build();

RealmConfiguration otherConfig = new RealmConfiguration.Builder()
  .name("otherrealm.realm")
  .schemaVersion(5)
  .modules(new MyOtherSchema())
  .build();

Realm myRealm = Realm.getInstance(myConfig);
Realm otherRealm = Realm.getInstance(otherConfig);
```

## Initialization


```java
@Override
public void onCreate() {
	super.onCreate();

	Realm.init(this);
	RealmConfiguration config = new RealmConfiguration.Builder().build();
	Realm.setDefaultConfiguration(config);
}
```

## Usage











```java
realm.beginTransaction();
User user = realm.createObject(User.class); // Create a new object
user.setName("John");
user.setEmail("john@corporation.com");
realm.commitTransaction();

User user = new User("John");
user.setEmail("john@corporation.com");

// Copy the object to Realm. Any further changes must happen on realmUser
realm.beginTransaction();
User realmUser = realm.copyToRealm(user);
realm.commitTransaction();
```





```java
realm.executeTransaction(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		User user = realm.createObject(User.class);
		user.setName("John");
		user.setEmail("john@corporation.com");
	}
});
```



```java
realm.executeTransactionAsync(new Realm.Transaction() {
	@Override
	public void execute(Realm bgRealm) {
		User user = bgRealm.createObject(User.class);
		user.setName("John");
		user.setEmail("john@corporation.com");
	}
}, new Realm.Transaction.OnSuccess() {
	@Override
	public void onSuccess() {
		// Transaction was a success.
	}
}, new Realm.Transaction.OnError() {
	@Override
	public void onError(Throwable error) {
		// Transaction failed and was automatically canceled.
	}
});
```




```java
@OnClick(R.id.tvAdd)
void add() {
	Realm mRealm = Realm.getDefaultInstance();

	// Create a new object
	mRealm.beginTransaction();
	for (int i = 0; i < 100; i++) {
		Person person = mRealm.createObject(Person.class, i);
		person.setName("shenhuniurou---" + i);
		person.setEmail("shenhuniurou@gmail.com---" + i);
	}
	mRealm.commitTransaction();
}

@OnClick(R.id.tvAsyncAdd)
void asyncAdd() {
	persons = new ArrayList<>();
	for (int i = 0; i < 100; i++) {
		Person person = new Person();
		person.setId(i + 100);
		person.setEmail("shenhuniurou@gmail.com---" + i);
		persons.add(person);
	}
	addPerson(persons);
}

private void addPerson(final List<Person> persons) {
	Realm mRealm = Realm.getDefaultInstance();

	addTask =  mRealm.executeTransactionAsync(new Realm.Transaction() {
		@Override
		public void execute(Realm realm) {
			for (Person person: persons) {
				realm.copyToRealm(person);
			}
		}
	}, new Realm.Transaction.OnSuccess() {
		@Override
		public void onSuccess() {
		}
	}, new Realm.Transaction.OnError() {
		@Override
		public void onError(Throwable error) {
			String message = error.getMessage();
		}
	});

}
```


```java
public void onStop () {
    if (transaction != null && !transaction.isCancelled()) {
        transaction.cancel();
    }
}
```


```java
public List<Person> queryAll() {
	Realm mRealm = Realm.getDefaultInstance();
	RealmResults<Person> persons = mRealm.where(Person.class).findAll();
	persons = persons.sort("id");
	return mRealm.copyFromRealm(persons);
}

public Person queryByCondition(String id) {
	Realm mRealm = Realm.getDefaultInstance();
	Person person = mRealm.where(Person.class).equalTo("id", id).findFirst();
	return person;
}

public void queryAll() {
	final Realm mRealm = Realm.getDefaultInstance();
	persons = mRealm.where(Person.class).findAllAsync();
	persons.addChangeListener(new RealmChangeListener<RealmResults<Person>>() {
		@Override
		public void onChange(RealmResults<Person> element) {
			element = element.sort("id");
			List<Person> personList = mRealm.copyFromRealm(element);
			mAdapter.addAll(personList);
			recyclerView.scrollToPosition(personList.size() - 1);
			persons.removeChangeListeners();
		}
	});
}
```

- between(),greaterThan(),lessThan(),greaterThanOrEqualTo()&lessThanOrEqualTo()
- equalTo() &?notEqualTo()
- contains(),?beginsWith() &?endsWith()
- isNull() &?isNotNull()
- isEmpty() &?isNotEmpty()


```java
RealmResults<Person> r = realm.where(Person.class)
                            .greaterThan("age", 10)  //implicit AND
                            .beginGroup()
                                .equalTo("name", "Peter")
                                .or()
                                .contains("name", "Jo")
                            .endGroup()
                            .findAll();
```


```java
RealmResults<User> results = realm.where(User.class).findAll();
long sum = results.sum("age").longValue();
long min = results.min("age").longValue();
long max = results.max("age").longValue();
double average = results.average("age");

long matches = results.size();
```



```java
Person item = mAdapter.getItem(position);
Realm  mRealm = Realm.getDefaultInstance();
Person person = mRealm.where(Person.class).equalTo("id", item.getId()).findFirst();
mRealm.executeTransaction(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		item.setName(person.getName());
		mAdapter.notifyItemChanged(position);
	}
});

Realm mRealm = Realm.getDefaultInstance();
updateTask = mRealm.executeTransactionAsync(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		Person person = realm.where(Person.class).equalTo("id", item.getId()).findFirst();
		item.setName(person.getName());
	}
}, new Realm.Transaction.OnSuccess() {
	@Override
	public void onSuccess() {
		mAdapter.notifyItemChanged(position);
	}
}, new Realm.Transaction.OnError() {
	@Override
	public void onError(Throwable error) {
	}
});

@Override
protected void onStop() {
	super.onStop();

	if (updateTask != null && !updateTask.isCancelled()) {
		updateTask.cancel();
	}
}
```


```java
Person item = mAdapter.getItem(position);
Realm  mRealm = Realm.getDefaultInstance();
Person person = mRealm.where(Person.class).equalTo("id", item.getId()).findFirst();
mRealm.executeTransaction(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		person.deleteFromRealm();
	}
});

Realm mRealm = Realm.getDefaultInstance();
deleteTask = mRealm.executeTransactionAsync(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		Person person = realm.where(Person.class).equalTo("id", item.getId()).findFirst();
		person.deleteFromRealm();
	}
}, new Realm.Transaction.OnSuccess() {
	@Override
	public void onSuccess() {
		mAdapter.remove(position);
	}
}, new Realm.Transaction.OnError() {
	@Override
	public void onError(Throwable error) {
	}
});

@Override
protected void onStop() {
	super.onStop();
	
	if (deleteTask != null && !deleteTask.isCancelled()) {
		deleteTask.cancel();
	}
}
```

