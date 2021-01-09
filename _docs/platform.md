---
title: Simpl Platform
permalink: /docs/platform/
layout: docs
description: Simpl Platform
---

## Simpl Platform

  <div class="container about">
    <div class="row">
      <div class="col-sm-12">
      <h5 class="u-title u-center u-spacerUp">Standard Simpl Framework</h5>
        <p class="text-center">The simpl-games-api is required to run one or more games. For each game, a model service and user interface (UI) will need to be created.
        </p>
      <section>
        <div class="row">
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M17 3H5c-1.11 0-2 .9-2 2v14c0 1.1.89 2 2 2h14c1.1 0 2-.9 2-2V7l-4-4zm-5 16c-1.66 0-3-1.34-3-3s1.34-3 3-3 3 1.34 3 3-1.34 3-3 3zm3-10H5V5h10v4z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-games-api</h5>
            <p class="tech__points--para">
              A single instance of simpl-games-api provides a database and REST API endpoints for one more more games. When data is changed, an event is sent to the registered game model service via an HTTP Webhook.
            </p>
          </div>
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zM6.5 9L10 5.5 13.5 9H11v4H9V9H6.5zm11 6L14 18.5 10.5 15H13v-4h2v4h2.5z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>Game Model Service</h5>
            <p class="tech__points--para">
              For each game, a model service must be created that provides communication between the simpl-games-api and UI.
            </p>
          </div>
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M22 18V3H2v15H0v2h24v-2h-2zm-8 0h-4v-1h4v1zm6-3H4V5h16v10z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>Game UI</h5>
            <p class="tech__points--para">
              For each game, a browser-based user UI must be created that communicates with the game model service. We recommend using simpl-ui-cookiecutter and the simpl-react package that comes pre-installed in it.
            </p>
          </div>
        </div>
      <h5 class="u-title u-center u-spacerUp">Creating a Game</h5>
        <p class="text-center">Use simpl-ui-cookiecutter and simpl-react to create a frontend. Choose from single of multi-player.
        </p>
        <div class="row">
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M12 6c1.11 0 2-.9 2-2 0-.38-.1-.73-.29-1.03L12 0l-1.71 2.97c-.19.3-.29.65-.29 1.03 0 1.1.9 2 2 2zm4.6 9.99l-1.07-1.07-1.08 1.07c-1.3 1.3-3.58 1.31-4.89 0l-1.07-1.07-1.09 1.07C6.75 16.64 5.88 17 4.96 17c-.73 0-1.4-.23-1.96-.61V21c0 .55.45 1 1 1h16c.55 0 1-.45 1-1v-4.61c-.56.38-1.23.61-1.96.61-.92 0-1.79-.36-2.44-1.01zM18 9h-5V7h-2v2H6c-1.66 0-3 1.34-3 3v1.54c0 1.08.88 1.96 1.96 1.96.52 0 1.02-.2 1.38-.57l2.14-2.13 2.13 2.13c.74.74 2.03.74 2.77 0l2.14-2.13 2.13 2.13c.37.37.86.57 1.38.57 1.08 0 1.96-.88 1.96-1.96V12C21 10.34 19.66 9 18 9z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-ui-cookiecutter</h5>
            <p class="tech__points--para">
              Built based on the popular cookiecutter library, a quickstart template to create a game UI
            </p>
          </div>
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M16.01 7L16 3h-2v4h-4V3H8v4h-.01C7 6.99 6 7.99 6 8.99v5.49L9.5 18v3h5v-3l3.5-3.51v-5.5c0-1-1-2-1.99-1.99z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-react</h5>
            <p class="tech__points--para">
              An npm module that provides support for building frontend UIs with React and Redux
            </p>
          </div>
          <div class="col-sm-4 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M9 8c1.11 0 2-.9 2-2s-.89-2-2-2c-1.1 0-2 .9-2 2s.9 2 2 2zm4 0c1.11 0 2-.9 2-2s-.89-2-2-2c-.36 0-.69.1-.98.27.3.51.48 1.1.48 1.73s-.18 1.22-.48 1.73c.29.17.63.27.98.27zM9 9.2c-1.67 0-5 .83-5 2.5V13h10v-1.3c0-1.67-3.33-2.5-5-2.5zM5 7H3V5H2v2H0v1h2v2h1V8h2V7zm9.23 2.31c.75.6 1.27 1.38 1.27 2.39V13H18v-1.3c0-1.31-2.07-2.11-3.77-2.39z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>Single or Multi-Player</h5>
            <p class="tech__points--para">
              Simpl includes support for single or multi-player games
            </p>
          </div>
          </div>
      <h5 class="u-title u-center u-spacerUp">Under the Hood</h5>
        <p class="text-center">Important Simpl packages that run in the background</p>
        <div class="row">
          <div class="col-sm-3 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M10 16v-1H3.01L3 19c0 1.11.89 2 2 2h14c1.11 0 2-.89 2-2v-4h-7v1h-4zm10-9h-4.01V5l-2-2h-4l-2 2v2H4c-1.1 0-2 .9-2 2v3c0 1.11.89 2 2 2h6v-2h4v2h6c1.1 0 2-.9 2-2V9c0-1.1-.9-2-2-2zm-6 0h-4V5h4v2z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-client</h5>
            <p class="tech__points--para">
              A Python client that provides asynchronous or synchronous data access between the simpl-modelservice and simpl-games-api
            </p>
          </div>
          <div class="col-sm-3 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zM6.5 9L10 5.5 13.5 9H11v4H9V9H6.5zm11 6L14 18.5 10.5 15H13v-4h2v4h2.5z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-modelservice</h5>
            <p class="tech__points--para">
              simpl-modelservice provides services common to all games, including maintaining a game's model data, making REST API calls to the simpl-games-api, and providing user data to the game user UI.
            </p>
          </div>
          <div class="col-sm-3 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M18 8h-1V6c0-2.76-2.24-5-5-5S7 3.24 7 6v2H6c-1.1 0-2 .9-2 2v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-1.1-.9-2-2-2zm-6 9c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2zm3.1-9H8.9V6c0-1.71 1.39-3.1 3.1-3.1 1.71 0 3.1 1.39 3.1 3.1v2z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-authenticator</h5>
            <p class="tech__points--para">
              A Web Application Messaging Protocol (WAMP) authenticator
            </p>
          </div>
          <div class="col-sm-3 u-center wow slideInUp" data-wow-offset="140">
            <div class="circle">
              <svg fill="#FFFFFF" id="gift" height="30" viewBox="0 0 24 24" width="30" xmlns="http://www.w3.org/2000/svg">
                  <path d="M9 8c1.66 0 2.99-1.34 2.99-3S10.66 2 9 2C7.34 2 6 3.34 6 5s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V16h14v-2.5c0-2.33-4.67-3.5-7-3.5z"/>
                  <path d="M0 0h24v24H0z" fill="none"/>
              </svg>
            </div>
            <h5>simpl-users</h5>
            <p class="tech__points--para">
                A custom Django User model tailored to Simpl's needs
            </p>
          </div>
          </div>
        <div class="row">
          <h5 class="u-title u-center u-spacerUp">Ways to Use Simpl</h5>
            <p class="text-center">As the Simpl ecosystem grows, options for using Simpl will grow</p>
            <p class="text-center">
              <ul>
                  <li>Create a game</li>
                  <li>Host a game</li>
                  <li>Use a game created by someone else</li>
              </ul>
            </p>
        </div>
      </section>
      </div>
    </div>
  </div>
