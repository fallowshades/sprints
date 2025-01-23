# Iteration 3 layer 2 Dashboard

[end-of-user-lifeCycle->navigation]

- remote reaction redirect
- context access to useNavigate (not for loading but logic)

## pre fetch before privilaged fetches

### entitled data boundary maintenance

#### 1. Get Current User

Each route can define a "loader" function to provide data to the route element before it renders.

- must return a value

DashboardLayout.jsx

```jsx
import { Outlet, redirect, useLoaderData } from 'react-router-dom';
import customFetch from '../utils/customFetch';

export const loader = async () => {
  try {
    const { data } = await customFetch('/users/current-user');
    return data;
  } catch (error) {
    return redirect('/');
  }
};


const DashboardLayout = ({ isDarkThemeEnabled }) => {
  const { user } = useLoaderData();

  return (
    <DashboardContext.Provider
      value={{
        user,
        showSidebar,
        isDarkTheme,
        toggleDarkTheme,
        toggleSidebar,
        logoutUser,
      }}
    >
      <Wrapper>
        <main className='dashboard'>
         ...
            <div className='dashboard-page'>
              <Outlet context={{ user }} />
            </div>
          </div>
        </main>
      </Wrapper>
    </DashboardContext.Provider>
  );
};
export const useDashboardContext = () => useContext(DashboardContext);
export default DashboardLayout;

```

main.jsx

```js
import { loader as dashboardLoader } from './pages/DashboardLayout'

     {
        path: '/dashboard',
        element: <DashboardLayout isDarkThemeEnabled={isDarkThemeEnabled} />,
        loader: dashboardLoader,
        children: [
          {
            ...
       } ]}
```

#### 2. Logout User

DashboardLayout.jsx

```js
import { useNavigate } from 'react-router-dom'
import { toast } from 'react-toastify'

const DashboardLayout = () => {
  const navigate = useNavigate()

  const logoutUser = async () => {
    navigate('/')
    await customFetch.get('/auth/logout')
    toast.success('Logging out...')
  }
}
```
