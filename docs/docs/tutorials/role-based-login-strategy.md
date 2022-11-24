To add role based authentication to your application, you must do three things.

1. Update your database schema
2. Add the `role` to the session object
3. Check for `role` in your pages/components

First modify the `user` table and add a `role` column with the type of `String?`.

Below is an example Prisma schema file.

```javascript title="/prisma/schema.prisma"
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  role          String?  // New Column
  accounts      Account[]
  sessions      Session[]
}

```

Next, implement a custom session callback in the `[...nextauth].js` file, as shown below.

```javascript title="/pages/api/auth/[...nextauth].js"
callbacks: {
  async session({ session, token, user }) {
    session.user.role = user.role; // Add role value to user object so it is passed along with session
    return session;
  }
},
```

Going forward, when using the `getSession` hook, check that `session.user.role` matches the required role. The example below assumes the role `'admin'` is required.

```javascript title="/pages/admin.js"
import { getSession } from "next-auth/react"

export default function Page({props}) {
    
  return(
  <div>
       Your Page data Here
  </div>
  )
}
export async function getServerSideProps(context){
  const session = await getSession(context)
   if (!session) {
    return {
      redirect: {
        destination: '/unauthorised',
        permanent: false,
      },
    }
  }

  if(session.user.role === "admin") {
  // fetch data and send the props or keep as it is
    return {
     props : {

     }
    }
  }
  else{
    return {
      redirect: {
        destination: '/unauthorised',
        permanent: false,
      },
    }
  }

}
```
Create a page unauthorised and redirect the user if session is not present or user is not authorised. Then it is up to you how you manage your roles, either through direct database access or building your own role update API.
