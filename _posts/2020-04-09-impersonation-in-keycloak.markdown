---
title: Impersonation in Keycloak
date: 2020-04-09 13:49:00 Z
categories:
- keycloak
tags:
- impersonation
---

I had to implement user impersonation using Keycloak. I haven't found any e2e solution so I decided to write short post about how we can achieve that.

First you need to start your Keycloak instance with these flags:
{% highlight properties %}
-Dkeycloak.profile=preview -Dkeycloak.profile.feature.token_exchange=enabled
{% endhighlight %}

It's allow us to use token exchange feature which is not enabled by default.
Then we need to enable impersonation for our client in Keycloak console.

![client.png](/uploads/client.png)

After that we need to grant impersonation for specified user:
![user.png](/uploads/user.png)

When we have configuration part done, we can move forward to implementation which is not much complex.

This is implementation of TypeScript service used in my Angular app for a token exchange.

{% highlight ts %}
export class ImpersonateService {
  constructor(private http: HttpClient) {}

  private static buildImpersonateRequestBody(userId: string, keycloakClientId: string, currentUserAccessToken: string): URLSearchParams {
    const body = new URLSearchParams();

    body.set('grant_type', 'urn:ietf:params:oauth:grant-type:token-exchange');
    body.set('subject_token', currentUserAccessToken);
    body.set('client_id', keycloakClientId);
    body.set('requested_subject', userId);

    return body;
  }

  private static getOptions() {
    return {
      headers: new HttpHeaders().append('Content-Type', 'application/x-www-form-urlencoded')
    };
  }

  public impersonate(keycloakUserService: string): Observable<any> {
    const body = ImpersonateService.buildImpersonateRequestBody(
      userId,
      this.config.config.oauth2.clientId,
      this.authStore.accessToken
    );

    // remember to replace master realm with your own
    return this.http
      .post(
        'http://localhost:8000/auth/realms/master/protocol/openid-connect/token',
        body.toString(),
        ImpersonateService.getOptions()
      )
      .pipe(
        tap(response => {
          // start using new token
          console.log(response['access_token']);
          console.log(response['refresh_token']);
        })
      );
  }
}
{% endhighlight %}

In response from Keyclok you should received new tokens pair.


Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!