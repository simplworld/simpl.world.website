---
layout: post
title: "Loading Run Data On Demand"
date: 2021-05-19 08:56:17 -0400
categories: Advanced
author: Jane Eisenstein
author_photo: /assets/img/authors/jane-eisenstein.png
featured: true
excerpt: |
    Leaders login more quickly when run data is loaded on demand.
---

When a user logs into a game, the Simpl frontend code figures out which runs the user is in and loads those runs' data into the redux store. 
For a user who is a leader in several runs, this can result in a large amount of data being loaded that user won't access.
Loading large amounts of run data can significantly delay the rendering of the leader's first page. 
A similar delay can occur if the user refreshes the browser window.

To handle such use cases, the `loadRunDataOnDemand` argument was added to the `simpl` decorator in version **0.8.0** of the `simpl-react` module. 

When the `loadRunDataOnDemand` argument is set to `true`, minimal run data is loaded into the redux store at login. 
This data comprises run objects and the user's runuser objects for those runs. 
This is sufficient data to support the leader selecting an individual run and then navigating to a run-specific page.


Any run-specific leader pages displaying player or world information need to load the run's full scope tree into the **simpl** redux state
when they are loaded into the browser. 
Once a run's full data has been loaded, it will remain in the redux store until it is unloaded. 
Leader pages that are not run-specific, can test for and unload run scope tree data from the **simpl** redux state.

This post will show you how to implement on demand run data loading in both multi-player and single-player games. 


## Loading Run Data On Demand in Multi-Player Games

First, configure the `loadRunDataOnDemand` argument of the `simpl` decorator in Root.js to load leader runs on demand:

```jsx
    export default simpl({
      authid: AUTHID,
      password: 'nopassword',
      url: `${MODEL_SERVICE}`,
      progressComponent: Progress,
      root_topic: ROOT_TOPIC,
      topics: () => topics,
      loadAllScenarios: false,
      loadRunDataOnDemand: LEADER,
    })(RootContainer);

```

Next, modify leader pages that are not run-specific to check for and unload run scope tree data when the page is loaded. For example:

```jsx
  ...

    class LeaderHome extends React.Component {  

      componentDidMount() {
        // unload any loaded worlds
        this.props.unloadRunDataAction();
      }
  
  ...

    const mapDispatchToProps = dispatch => {
      return {
        unloadRunDataAction() {
          dispatch(SimplActions.unloadRunData());
        }
      }
    };

```

Last, modify all run-specific leader pages that display player or world data to load the run's scope tree data if it's not already loaded. For example:

```jsx
    ...

    class LeaderRunDebrief extends React.Component {
    
      componentDidMount() {
        // load run's worlds if not already loaded
        const {run, loadedRunId, loadRunDataAction} = this.props;
        loadRunDataAction(run, loadedRunId);
      }

    ...
      
    function mapStateToProps(state, ownProps) {
      const run = state.simpl.run.find(
        (r) => r.id == ownProps.params['id']
      );

      const unsortedWorlds = state.simpl.world.filter(
        (w) => run.id === w.run
      );
      const worlds = _.sortBy(unsortedWorlds, (s) => s.id);   // worlds are created in order

      return {
        run,
        worlds,
        loadedRunId: state.simpl.loaded_run // non-null if run data has been loaded
      };
    }

    const mapDispatchToProps = dispatch => {
      return {
        loadRunDataAction(run, loadedRunId) {
          if (!isNil(run)) {
            if (run.id !== loadedRunId) {
              dispatch(SimplActions.loadRunData(run.id)); // does not load player scenarios
            }
          }
        },
      };
    };

```

The `SimplActions.loadRunData` has a second parameter `loadPlayerScenarios` that defaults to false. 
Omitting it prevents player scenarios from being loaded with the other run data.

These changes are implemented in the `simpl-div-ui` repository. 


## Loading Run Data On Demand in Single-Player Games


First, configure the `loadRunDataOnDemand` argument of the `simpl` decorator in Root.js to load leader runs on demand. For example:

```jsx
    export default simpl({
      authid: AUTHID,
      password: 'nopassword',
      url: `${MODEL_SERVICE}`,
      progressComponent: Progress,
      root_topic: ROOT_TOPIC,
      topics: () => topics,
      loadAllScenarios: LEADER,
      loadRunDataOnDemand: LEADER,
    })(RootContainer);

```

Note the `loadAllScenarios` argument will be `true` for leaders, so they can access player scenarios.


Next, modify leader pages that are not run-specific to check for and unload run scope tree data when the page is loaded. 
This code is the same in both multi-player and single-player games. 

```jsx
  ...

    class LeaderHome extends React.Component {  

      componentDidMount() {
        // unload any loaded worlds
        this.props.unloadRunDataAction();
      }
  
  ...

    const mapDispatchToProps = dispatch => {
      return {
        unloadRunDataAction() {
          dispatch(SimplActions.unloadRunData());
        }
      }
    };

```

Last, modify all run-specific leader pages that display player data to load the run's scope tree data if it's not already loaded. For example:

```jsx
    ...

    class LeaderRun extends React.Component {
    
      componentDidMount() {
        // load run's players if not already loaded
        const {run, loadedRunId, loadRunDataAction} = this.props;
        loadRunDataAction(run, loadedRunId);
      }

    ...

      function mapStateToProps(state, ownProps) {
      const run = state.simpl.run.find(
        (r) => r.id == ownProps.params['id']
      );

      const unsortedPlayers = state.simpl.runuser.filter(
        (ru) => ru.run === run.id && ru.leader === false
      );
      const players = _.sortBy(unsortedPlayers, (p) => p.email);

      return {
        run,
        players,
        loadedRunId: state.simpl.loaded_run // non-null if run data has been loaded
      };
    }

      const mapDispatchToProps = dispatch => {
      return {
        loadRunDataAction(run, loadedRunId) {
          if (!isNil(run)) {
            if (run.id !== loadedRunId) {
              dispatch(SimplActions.loadRunData(run.id, true)); // load player scenarios
            }
          }
        },
      };
    };

```

Here the `SimplActions.loadRunData` second parameter is set to `true` to ensure the player scenarios are loaded into the redux store.

These changes are implemented in the `simpl-calc-ui` repository.

## Running the Example Games with Multiple Game Runs

Both these example game front ends can work with more than one game run.

Their accompanying model services (`simpl-div-model` and `simpl-calc-model`) support creating 
runs with non-default names. For example, running:

```shell
./manage.py create_default_env -n a

```
will create a run named **a** with players whose email addresses start with 'a'.

## Summary

Loading run data on demand can speed up rendering of leader pages. 
Not all games will need this feature. It can be added late in the development cycle if 
leaders with large numbers of runs experience sluggish page rendering.















