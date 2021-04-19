# React To Create a new Store
## Zsh script - rxjs, @reduxjs/toolkit, redux-observable, ts

You can just add this script into .zshrc.

You can create a new file (i.e. .zshrc_react_snippet) and add into .zshrc one:
`source ~/.zshrc_react_snippet`

```
#!bin/zsh

reactNewStore () {
    path=$PWD
    storeName=$1
    dir="$(/usr/bin/basename $storeName)" 
    echo "${path} perform new store ${1}"
    /bin/mkdir ${storeName}
    /bin/cat <<EOT >> ${storeName}/${dir}.epic.ts
import { PayloadAction } from '@reduxjs/toolkit';
import { catchError, map, mergeMap } from 'rxjs/operators';
import { Action } from 'redux';
import { Observable, of } from 'rxjs';
import { Epic, ofType } from 'redux-observable';
import { ${dir}Actions } from './${dir}.slice';

const ${dir}RequestEpic: Epic =
  (action$: Observable<PayloadAction>): Observable<Action> => {
    return action$.pipe(
      ofType(${dir}Actions.${dir}Request.type),
      mergeMap((action) => Services.api(action.payload)),
      map((response: Models) => ({ type: ${dir}Actions.${dir}Success.type, payload: response })),
      catchError((error: Error) => of({ type: ${dir}Actions.${dir}Failure.type, payload: error }))
    )
  }

const epics = [
  ${dir}RequestEpic,
];

export default epics;
EOT

  /bin/cat <<EOT >> ${storeName}/${dir}.selectors.ts
import { createSelector } from 'reselect';
import { AppState } from './../index';

const getPending = (state: AppState) => state.${dir}.loading;
const getError = (state: AppState) => state.${dir}.error;

const ${dir} = (state: AppState) => state.${dir}.data;

export const getPendingSelector = createSelector(getPending, (pending) => pending);
export const getErrorSelector = createSelector(getError, (error) => error);
export const ${dir}Selector = createSelector(${dir}, (data) => data);
EOT

  /bin/cat <<EOT >> ${storeName}/${dir}.types.ts
import { AxiosError } from 'axios';

export interface ${dir}State {
  data: Models,
  error?: AxiosError,
  loading?: boolean,
}
EOT

  /bin/cat <<EOT >> ${storeName}/${dir}.slice.ts
import { createSlice } from '@reduxjs/toolkit'
import { ${dir}State } from './${dir}.types';

const initialState: ${dir}State = {
  data: Models,
}
export const ${dir}Slice = createSlice({
  name: '${dir}',
  initialState,
  reducers: {
    ${dir}Request: (state, action) => {
      state.loading = true;
      state.filter = action.payload;
    },
    ${dir}Success: (state, action) => {
      state.loading = false;
      state.data = action.payload;
    },
    ${dir}Failure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    }
  },
})

export const ${dir}Actions = ${dir}Slice.actions;
export const ${dir}Reducer = ${dir}Slice.reducer;
EOT
}
```
