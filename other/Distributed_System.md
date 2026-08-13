// Bismillah
//* B_Hypercarp_and_the_Control_Panel.cpp   13-08-2026  21:24:04
//* Artist: Asaduzzaman Alamin
#include <bits/stdc++.h>
#define ll long long int
#define endl '\n'
using namespace std;
void ART()
{
    ll n;
    cin >> n;
    ll ANS = 0;
    vector<ll> x(n);
    map<ll, ll> mp;
    for (int i = 0; i < n; ++i)
        cin >> x[i], ++mp[x[i]];
    vector<ll> temp;
    if (mp.size() == 1)
    {
        cout << 1 << endl;
        return;
    }
    if (n == 1)
    {
        cout << 1 << endl;
        return;
    }
    if (n == 2)
    {
        if (x[0] != x[1])
        {
            cout << 2 << endl;
            return;
        }
        else
        {
            cout << 1 << endl;
            return;
        }
    }
    for (int i = 0; i < n; ++i)
    {
        ll Cnt = 0;
        ll j = i;
        ll task = x[i];
        while (x[j] == x[i])
            ++Cnt, ++j;
        if (Cnt >= 2)
            temp.push_back(x[i]), temp.push_back(x[i]);
        else
            temp.push_back(x[i]);
        i = j - 1;
        ++ANS;
    }
    bool flag = false;
    for (int i = 0; i < temp.size() - 3; ++i)
    {
        if (temp[i] == temp[i + 1] and temp[i + 1] != temp[i + 2] and temp[i + 2] == temp[i + 3])
        {
            flag = true;
            break;
        }
    }
    if (flag)
    {
        cout << ANS + 2 << endl;
        return;
    }
    auto check = [](vector<ll> &a)
    {
        ll result = 0;
        for (int i = 0; i < a.size(); ++i)
        {
            // if(a[i] ==  a[i+1] and  a[i+1] !=  a[i+3]){
            //     result  = 1;
            //     break;

            // }
            if (i + 2 < a.size() and a[i + 1] == a[i] and a[i + 1] != a[i + 2])
            {
                result = 1;
                break;
            }
        }

        for (int i = 0; i < a.size(); ++i)
        {
            if (i + 3 < a.size() and a[i] == a[i + 1] and a[i + 1] != a[i + 3])
            {
                result = 1;
                break;
            }
        }

        return result;
    };

    ll take_ans = check(temp);
    reverse(temp.begin(), temp.end());
    ll take_ans2 = check(temp);
    cout << ANS + max(take_ans2, take_ans) << endl;
}
int main()
{
    // 如果心中充满信念，无比坚定，就一定会看到胜利。
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    ll tt = 1;
    cin >> tt;
    for (ll i = 1; i <= tt; ++i)
    {
        // cout << "Case #" << i << ": ";
        ART();
    }
    return 0;
}
// Alhamdulillah