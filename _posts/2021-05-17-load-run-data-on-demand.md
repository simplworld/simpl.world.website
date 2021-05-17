---
layout: post
title: "Loading Run Data On Demand"
date: 2021-05-17 08:56:17 -0400
categories: Advanced
author: Jane Eisenstein
author_photo: /assets/img/authors/jane-eisenstein.png
featured: true
excerpt: |
    Leaders login more quickly when run data is loaded on demand.
---

When a user logs into a game, the Simpl frontend code figures out which runs the user is in and loads those runs' data into the redux store. 
For a user who is a leader in several runs, this can result in a large amount of data being loaded that user never accesses. 
Loading large amounts of run data, can significantly delay the rendering of the leader's first page.

To handle such use cases, the `loadRunDataOnDemand` argument was added to the `simpl` decorator in `simpl-react` version **0.8.0**. 
When the `loadRunDataOnDemand` argument is set to `true`, minimal run data is loaded at login. 
Rather, run-specific leader pages load the run's full scope tree when the user navigates to them. 
Once a run's full data has been loaded into the **simpl** redux state it remains in the redux store until the user navigates back to a page that is not run-specific. 
When a leader page loads that is not run-specific, it unloads any run scope tree data from the **simpl** redux state.

This post will show you how to implement on demand run data loading. 


## Loading Run Data On Demand in Multi-Player Games

First, configure the `loadRunDataOnDemand` argument of the simpl decorator in Root.js to load leader runs on demand:

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

Next, modify leader pages that are not run-specific to check for and unload run scope tree data when the page is loaded.

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

Last, modify all run-specific leader pages to load the run's scope tree data if it's not already loaded.

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

These changes are implemented in the `load-run-data-on-demand` branch of the `simpl-div-ui` repository. 

To work with more than one **Simpl Div** game run, use the `profiling` branch of the `simpl-div-model` repository. 
This branch supports creating runs with non-default names.

```shell
./manage.py create_default_env -n div

```
will create a run named **div** with players whose email addresses start with 'div'.

## Loading Run Data On Demand in Single-Player Games



## Summary















