# React + Typescript + Redux Toolkit
### Step 1
1. `npx create-react-app myapp --template typescript`
2. `cd myapp`
3. `npm install -D prettier eslint-config-prettier eslint-plugin-prettier`
4. setup package.json
    1. under "scripts" add 
        ```json
        {
            "scripts": {
                "lint": "eslint --ext .js,.jsx,.ts,.tsx src"
            }
        }
        ```
    2. under "eslintConfig"
        ```json
        {
            "eslintConfig": {
                "extends": [
                    "eslint-config-react-app",
                    "plugin:eslint-plugin-prettier/recommended"
                ]
            }
        }
        ```
    3. create "prettier" key
        ```json
        {
            "prettier": {
                "printWidth": 80,
                "singleQuote": true,
                "trailingComma": "none",
                "tabWidth": 2,
                "arrowParens": "avoid",
                "semi": false
            }
        }
        ```
    4. create .vscode/settings.json
        ```json
        {
            "editor.formatOnSave": false,
            "editor.codeActionsOnSave": {
                "source.fixAll.eslint": true 
            }
        }
        ```

### Step 2
1. `npm install @reduxjs/toolkit react-redux @types/react-redux axios`
2. create rootReducer
    ```typescript
    // src/app/rootReducer.ts
    import { combineReducers } from '@reduxjs/toolkit'

    const rootReducer = combineReducers({})

    export type RootState = ReturnType<typeof rootReducer>
    export default rootReducer
    ```
3. create store
    ```typescript
    // src/app/store.ts
    import { configureStore, ThunkAction, Action } from '@reduxjs/toolkit'
    import rootReducer, { RootState } from './rootReducer'
    
    const store = configureStore({
      reducer: rootReducer
    })
    
    export type AppDispatch = typeof store.dispatch
    export type AppThunk = ThunkAction<void, RootState, unknown, Action<string>>
    export default store
    ```
4. Hooked it to the main index file
    ```jsx
    // src/index.tsx
    import React from 'react'
    import ReactDOM from 'react-dom'
    import { Provider } from 'react-redux'
    import store from './app/store'
    import App from './app/App'
    
    const render = () => {
      ReactDOM.render(
        <Provider store={store}>
          <App />
        </Provider>,
        document.getElementById('root')
      )
    }
    
    render()
    ```

### Step 3
1. fetch something
    ```typescript
    // src/api/githubAPI.ts
    import axios from 'axios'

    export interface RepoDetails {
      id: string
      name: string
    }
    
    export async function getRepoDetails(org: string, repo: string) {
      const url = `https://api.github.com/repos/${org}/${repo}`
    
      const { data } = await axios.get<RepoDetails>(url)
      return data
    }
    ```
2. create slice
    ```typescript
    // src/features/repoDetails/repoDetailsSlice.ts
    import { createSlice, PayloadAction } from '@reduxjs/toolkit'

    import { AppThunk } from '../../app/store'
    
    import { getRepoDetails, RepoDetails } from '../../api/githubAPI'
    
    interface RepoDetailsState {
      id: string
      name: string
      error: string | null
    }
    
    const initialState: RepoDetailsState = {
      id: '',
      name: '',
      error: null
    }
    
    const repoDetailsSlice = createSlice({
      name: 'repoDetails',
      initialState,
      reducers: {
        getRepoDetailsSuccess(state, action: PayloadAction<RepoDetails>) {
          state.id = action.payload.id
          state.name = action.payload.name
          state.error = null
        },
        getRepoDetailsFailed(state, action: PayloadAction<string>) {
          state.error = action.payload
        }
      }
    })
    
    export const {
      getRepoDetailsSuccess,
      getRepoDetailsFailed
    } = repoDetailsSlice.actions
    
    export default repoDetailsSlice.reducer
    
    export const fetchRepoDetails = (
      org: string,
      repo: string
    ): AppThunk => async dispatch => {
      try {
        const result = await getRepoDetails(org, repo)
        dispatch(getRepoDetailsSuccess(result))
      } catch (err) {
        dispatch(getRepoDetailsFailed(err.toString()))
      }
    }
    ```
3. attach it to the rootReducer
    ```typescript
    // src/app/rootReducer.ts
    import { combineReducers } from '@reduxjs/toolkit'
    
    import repoDetailsReducer from '../features/repoDetails/repoDetailsSlice'
    
    const rootReducer = combineReducers({
      repoDetails: repoDetailsReducer
    })
    
    export type RootState = ReturnType<typeof rootReducer>
    export default rootReducer
    ```
4. hooked it to a component
    ```jsx
    // src/app/App.tsx
    import React, { useEffect } from 'react'
    import { useSelector, useDispatch } from 'react-redux'
    import { fetchRepoDetails } from '../features/repoDetails/repoDetailsSlice'
    import { RootState } from './rootReducer'
    
    const App = () => {
      const dispatch = useDispatch()
      const { id, name } = useSelector((state: RootState) => state.repoDetails)
    
      useEffect(() => {
        dispatch(fetchRepoDetails('reduxjs', 'redux'))
      })
      
      return (
        <div>
          <p>id:{id}</p>
          <p>name:{name}</p>
        </div>
      )
    }

    export default App
    ```
