## Introduktion/Bakgrund

Som jag nämnde i inlämning del 1 så antog vi där att en färdig MVC-applikation redan existerade innan vi satte upp Docker Swarm-infrastrukturen. I verkligheten är detta inte det mest praktiska arbetsflödet.

I denna rapport (del 2) utforskar jag det alternativa tillvägagångssättet där frontend och backend separeras, med frontend hostad på AWS S3. Detta möjliggör snabba uppdateringar av användargränssnittet utan container-rebuilds, vilket är mer lämpat för aktiv utveckling.