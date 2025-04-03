There is a website with 4 links to hawk pictures :

```html
<a href="/?file=assets/data/img/cooper-s-hawk-profile-583855629-d89e191a88d1484db08800f067ba98e8.jpg&hash=$2a$12$NmPFGriPq4VEFdx7y4XKde67/DFQgQVk/Cz.HxGWi0PV3aSk/JT12">Hawk1</a>
<a href="/?file=assets/data/img/harris_hawk_web.jpg&hash=$2a$12$hPNstQ8F.EBu8z2/EDaXROPakN5L/hix0SUQQG6I6RPu/BcvSBDmC">Hawk2</a>
<a href="/?file=assets/data/img/Hawk-146809760-612x612.jpg&hash=$2a$12$dp0lDL1FuN6irg2LB7j.EOFKte1313GSgz5DpBeTAtBY4gyCMd4KS">Hawk3</a>
<a href="/?file=assets/data/img/Hawk-534214314-612x612.jpg&hash=$2a$12$5zq3d97d5wg1vUvoquZOA.JAeM1.778eWnDgSx/ymj9v1D8d4kLEC">Hawk4</a>
<a href="/?file=assets/data/img/Red-shouldered_Hawk_Buteo_lineatus_-_Blue_Cypress_Lake_Florida.jpg&hash=$2a$12$qsfcrstGdzpRVDNH5Dq//uSK6/Z6ZSBCca7fIeoyRBBdgQk8q3rX6">Hawk5</a>
```

There is a file and hash parameter sent along with the request, wich are hard coded :

```html
// Authorized images are matched to their bcrypt hash values for maximum security
$AUTHORIZED_IMGS = [
    '$2a$12$NmPFGriPq4VEFdx7y4XKde67/DFQgQVk/Cz.HxGWi0PV3aSk/JT12' => 'assets/data/img/cooper-s-hawk-profile-583855629-d89e191a88d1484db08800f067ba98e8.jpg',
    '$2a$12$qsfcrstGdzpRVDNH5Dq//uSK6/Z6ZSBCca7fIeoyRBBdgQk8q3rX6' => 'assets/data/img/Red-shouldered_Hawk_Buteo_lineatus_-_Blue_Cypress_Lake_Florida.jpg',
    '$2a$12$hPNstQ8F.EBu8z2/EDaXROPakN5L/hix0SUQQG6I6RPu/BcvSBDmC' => 'assets/data/img/harris_hawk_web.jpg',
    '$2a$12$dp0lDL1FuN6irg2LB7j.EOFKte1313GSgz5DpBeTAtBY4gyCMd4KS' => 'assets/data/img/Hawk-146809760-612x612.jpg',
    '$2a$12$5zq3d97d5wg1vUvoquZOA.JAeM1.778eWnDgSx/ymj9v1D8d4kLEC' => 'assets/data/img/Hawk-534214314-612x612.jpg', 
    //'$2a$12$v5UW4B3/j6F5vymG0tRDx.iSz7RFlrVlH3Om3zC3QfqiG.InCuKMW' => 'flag.txt'
];
```

When doing a request, the parameters are checked against the values of the $AUTHORIZED_IMGS variable.

```php
if (isset($AUTHORIZED_IMGS[$provided_hash]) && password_verify($file_name, $provided_hash)) {
    header("Content-Type: image/png");
    echo readfile($file_name); // <-- LFI HERE
    exit();
}
```

With the LFI i tried to directly access the flag : `hash=$2a$12$v5UW4B3/j6F5vymG0tRDx.iSz7RFlrVlH3Om3zC3QfqiG.InCuKMW&file=flag.tx`

The challenge refere to Okta and recent vuln on okta is the wrong usage of `bcrypt` wich only cares about the first 72 bytes. 
So using password_verify with the hash being, for example, the hash of 72 times A and the plaintext to verify against being 72 times A and then anything you want, will always return 1 (so valid). 

Looking at the file names there are two that are over 72 characters long:

```
    assets/data/img/cooper-s-hawk-profile-583855629-d89e191a88d1484db08800f067ba98e8.jpg
    assets/data/img/Red-shouldered_Hawk_Buteo_lineatus_-_Blue_Cypress_Lake_Florida.jpg
```

We can take one of them, pass the hash=<hash> and append `../../../../../flag.txt` to the file parameter to get the content of the flag.txt file.

`INS{**_FLAG_**}`
