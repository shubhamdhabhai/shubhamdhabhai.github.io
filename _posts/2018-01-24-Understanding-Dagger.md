---
layout: post
---
## Dagger 2
Dagger 2 is a library for Dependency Injection in java. There are other libraries also like *RoboGuice* but the most popular and effective is Dagger. You may ask why?  
Because it generates all the required code and does not use any reflection, which is slow in general. This generate-all-the-code approach has the advantage that any mistake regarding the dependency graph will be caught during compile-time instead of run-time.

There are many article written on it and I am not going to write the same thing. This blog mainly deals with dependent components.  

Some of the sources are   

[A superb sample covering real world scenerio](https://github.com/abhishekBansal/android-mvp-retrofit2-dagger2-rxjava2-testing).  
 [Dependency Injection with Dagger 2](https://guides.codepath.com/android/dependency-injection-with-dagger-2).  
 [An  Introduction to Dagger 2](https://dzone.com/articles/an-introduction-to-dagger-2-android-di-part-1-3).  
 [Dagger 2. Part II. Custom scopes, Component dependencies, Subcomponents](https://proandroiddev.com/dagger-2-part-ii-custom-scopes-component-dependencies-subcomponents-697c1fa1cfc).

### Dependent Components
There are two ways in which we can pass the dependency from one component to another, **SubComponent** and **Dependent Component**, in this blog I will discuss about dependent Components with the help of a [Github sample](https://github.com/shubhamdhabhai/Dagger-MVP)

### About the sample
This sample covers a real word scenario in which I have used some libraries like **retrofit**, **Butterknife**, **Lombok** etc which I think every android developer should use. I have also used **Model-View-Presenter** architecture because it compliments **Dagger**. I had to stop myself from using **RxJava** because I did not want **RxJava** to take the limelight from **Dagger**. I will write a blog specially for **RxJava** latter.  
This sample uses two api sources one is get the list of repo for a particular user and another one gets List of jobs. These are open apis so we do not need authentication for this. On the home screen there are two buttons, *Get Github Repo List*  takes us to the screen to get public repo list of the username we enter and *Get job list* takes us to the screen where on clicking *Get Jobs* it fetches the list of jobs.
Simple enough right.

I have divided the app in 5 scopes,
#### Singleton
This is the application wide scope which includes entities like OkHttpClient and GsonConverterFactory. We want them to be created once for the whole application and use them through out the app.

#### GithubApiScope and JobApiScope
We have two api different api sources, one is [Github](https://developer.github.com/v3/repos/#list-user-repositories) and another is for getting [list of jobs](https://github.com/workforce-data-initiative/skills-api/wiki/API-Overview). So I have created separate scopes for them.

#### RepoListScope and JobListScope
We have a home screen in our app from which we can either navigate to list of repos or list of jobs. Since they are entirely separate functionality I have created separate scopes for them.  
To show how these scopes depend on each other I will use a diagram.
![Alt text](/dagger_diagram.png)

You can see that GithubApiScope and JobApiScope depends on Singleton scope and RepoListScope depends on RepoListScope and JobListScope depends on JobApiScope.  

#### Lets write the code
First I have created a AppModule which takes care of providing **OkHttpClient** and **GsonConverterFactory** to the complete application. They will be useful when I create service for Github api and job api.  
```
@Module
public class AppModule {

    @Provides
    @Singleton
    public OkHttpClient providesOkHttpClient(HttpLoggingInterceptor interceptor) {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        ...
        return builder.build();
    }

    @Provides
    @Singleton
    public GsonConverterFactory providesGsonConvertorFactory() {
        Gson gson = new GsonBuilder()
                .setDateFormat("yyyy-MM-dd")
                .create();
        return GsonConverterFactory.create(gson);
    }
}
```

Then I have to create a component for this **AppModule**

```
@Singleton
@Component(modules = {AppModule.class})
public interface AppComponent {
    OkHttpClient getOkHttpClient();
    GsonConverterFactory getGsonConvertorFactory();
}
```

Note that I have provided functions which return **OkHttpClient** and **GsonConverterFactory**. I have to write functions in parent component that return the object that are needed in the child component. If these functions are not written then we cannot access them, Also note that I have not used any inject method in this as I can delegate that work to child component.
This gives us control of what I want to expose to the child as opposed to **Subcomponent** where everything in the parent is exposed to child.

Next I have created Module and Component for Github Api Service.

```
@Module
public class GithubApiModule {

    @GithubApiScope
    @Provides
    public GitHubApiService provideGithubApiService(OkHttpClient client, GsonConverterFactory gsonConverterFactory) {

        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.github.com/")
                .client(client)
                .addConverterFactory(gsonConverterFactory)
                .build();
        return retrofit.create(GitHubApiService.class);
    }
}
```

```
@GithubApiScope
@Component(dependencies = {AppComponent.class}, modules = {GithubApiModule.class})
public interface GithubApiComponent {
    GitHubApiService getGithubApiService();
}
```

In the **GithubApiModule** I are using **OkHttpClient** and **GsonConverterFactory** which I created in AppComponent. This is posiible because I have added AppComponent as dependency to GithubApiComponent.  
Also notice that I have used a different scope **@GithubApiScope** for this as I will be using this only in that part of the application where I need GitHubApi and not anywhere else.  
I also created a function **getGithubApiService()** in GithubApiComponent so that the child component can get access to GitHubApiServiceal. No **inject** method in **GithubApiComponent** also. Remember I want to delegate the work of injection to its child.  

Next I have created the **Module** and **Component** for our activity which uses **GitHubApiService**.

```
@Module
public class RepoListModule {
    private RepoListContract.IRepoListView repoListView;

    public RepoListModule(RepoListContract.IRepoListView view) {
        repoListView = view;
    }

    @Provides
    @RepoListScope
    public RepoListContract.IRepoListPresenter providesRepoListPresenter(GitHubApiService apiService) {
        return new RepoListPresenter(repoListView, apiService);
    }

    @Provides
    @RepoListScope
    public RepoListAdapter providesRepoAdapter() {
        return new RepoListAdapter();
    }

}
```

```
@RepoListScope
@Component(dependencies = {GithubApiComponent.class}, modules = {RepoListModule.class})
public interface RepoListComponent {

    void inject(RepoListActivity repoListActivity);
}
```

Here I am again using a different scope as this scope, **@RepoListScope**, which is only for this particular screen(activity or fragment).  
The module needs the **GitHubApiService**, which it will get from **GithubApiComponent**. It also provides the **RepoListAdapter** which is useful for the recycler view in this activity.  
The **RepoListComponent** does not contain any method which provides any object as there is no child component for this one. It only have the inject method.  

Now all I need to do is initialise the components and inject the dependency.

```

public class BaseApplication extends Application {

    private AppComponent appComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        appComponent = DaggerAppComponent.create();
    }

    public AppComponent getAppComponent() {
        return appComponent;
    }
}


public class RepoListActivity extends BaseActivity implements RepoListContract.IRepoListView {
  ...
  @Inject
    RepoListContract.IRepoListPresenter repoListPresenter;

    @Inject
    RepoListAdapter repoListAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        GithubApiComponent githubApiComponent = DaggerGithubApiComponent.builder()
                .appComponent(((BaseApplication) getApplication()).getAppComponent())
                .githubApiModule(new GithubApiModule())
                .build();

        RepoListComponent repoListComponent = DaggerRepoListComponent.builder()
                .githubApiComponent(githubApiComponent)
                .repoListModule(new RepoListModule(this))
                .build();
        repoListComponent.inject(this);
        ...
    }
    ...
}

```
In my application class I have created **AppComponent**. I can use **DaggerAppComponent.create();** because my AppComponent does not requires anything to initialise. Then in the **onCreate** method of my activity I have first initialised **GithubApiComponent** which required **AppComponent** as it depends on it. Then I have initialised  **RepoListComponent**.

I could have done this initialisation in **BaseApplication** and got **RepoListComponent** from there as I am getting **AppComponent** but then I would have to take responsibility of creating and destroying it in my activities onCreate and onDestroy method.  

Lastly I called the **inject** method to tell **RepoListComponent** that this activity requires some dependencies.
Since I needed **RepoListAdapter** and **RepoListContract.IRepoListPresenter** in this activity so I injected them by using **@Inject** annotation.  

Now lets take a look at how I am getting the list of repos in my presenter.

```

public class RepoListPresenter implements RepoListContract.IRepoListPresenter {

    private final RepoListContract.IRepoListView view;
    private final GitHubApiService githubApiService;

    public RepoListPresenter(RepoListContract.IRepoListView view, GitHubApiService githubApiService) {
        this.view = view;
        this.githubApiService = githubApiService;
    }

    @Override
    public void getGithubRepoList(String username) {
        githubApiService.getRepoFromUserName(username).enqueue(new Callback<List<Repo>>() {
            @Override
            public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
                view.onGetRepoListSuccess(response.body());
            }

            @Override
            public void onFailure(Call<List<Repo>> call, Throwable t) {
                view.onGetRepoListFailure(t);
            }
        });
    }
}
```

I do not need to do anything related to Dagger here as **RepoListModule** takes care of creating **RepoListPresenter** and **RepoListComponent** takes care of passing that to **RepoListActivity**.  
We need to follow the same steps for gettng jobs list as well. You can try that yourself and if you face any problem refer to my [github sample]((https://github.com/shubhamdhabhai/Dagger-MVP)).  

That's it, we have created an application which is modular, scalable and very easy to write unit test cases.  

May the force be with you!!
